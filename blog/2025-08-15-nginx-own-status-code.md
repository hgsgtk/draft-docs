# Understanding Standard and Nginx-Specific HTTP Status Codes for SREs

This article covers both common standard status codes and unique Nginx-specific ones—so you’ll know exactly what you’re seeing in your logs.

## Overview

For SREs operating web applications, understanding common HTTP status codes is critical. RFC 9110 defines standard HTTP response status codes, but you’ll see unlisted HTTP status codes when you operate Nginx servers. This is because Nginx defines non-standard HTTP status codes for logging and dynamic controls. This blog introduces those unique HTTP status codes in addition to the typical standard HTTP response status codes.


## Standard HTTP Status Codes

RFC 9110 HTTP Semantics defines [HTTP response status codes](https://httpwg.org/specs/rfc9110.html#overview.of.status.codes). All valid status codes are within[the range of 100 to 599](https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml), inclusive. 

* 1xx (Informative): 100-104
    * e.g., 100 Continue, 101 Switching Protocols
    * Unassigned 105-199 


* 2xx (Successful): 200-226 
    * e.g., 200 OK, 201 Created, 202 Accepted
    * Unassigned 209-225, 227-299

* 3xx (Redirection): 300-308 
    * e.g., 301 Moved Permanently, 304 Not Modified, 307 Temporary Redirect
    * Unassigned 309-399

* 4xx (Client Error): 400-418, 421-431, 451
    * e.g., 400 Bad Request, 401 Unauthorized, 404 Not Found
    * Unassigned 419-420, 432-450, 452-499

* 5xx (Server Error): 500-511
    * e.g., 500 Internal Server Error, 502 Bad Gateway, 504 Gateway Timeout
    * Unassigned 512-599

Looking at typical error cases, 500 Internal Server Error indicates that the server encountered an unexpected condition. This error is a general “catch-all” response to server issues. 

502 (Bad Gateway) is also a generic “catch-all”, but different from 500, in that the error has occurred in the network flow, involving a gateway or proxy. The error is returned when a gateway or proxy receives an invalid response from the upstream server. If they do not receive any HTTP response from the origin, a client gets a 504 Gateway Timeout response instead.

## Nginx's own HTTP status codes

Nginx defines extra [HTTP response status codes](https://http-statuscode.com/en/code/4XX) in [ngx_http_request.h](https://github.com/nginx/nginx/blob/251444fcf4434bfddbe3394a568c51d4f7bd857f/src/http/ngx_http_request.h#L136) to handle specific scenarios. These won’t appear in the official RFC but may show up in your logs.

* 444 No Response
    * Close the connection without sending a response to the client, most commonly used to deny malicious or malformed requests.


* 494 Request Header Too Large
    * The client has either sent a too large HTTP request or the transmitted HTTP headers are too long.

* 495 SSL Certificate Error
    * An error occurred during the verification of the client certificate.

* 496 SSL Certificate Required
    * Request that does not contain an SSL certificate, although one is required.

* 497 HTTP Request Sent to HTTPS Port
    * A regular HTTP request was sent to the HTTPS port.

* 499 Client Closed Request
    * The client closed the request before the server could send a response.

## Example: Triggering a 444 No Response

You can test the 444 behavior with the following Nginx config:

```
http {
  # HTTP server (code 444)
  server {
    listen 80;
    server_name localhost;

    location /test_444 {
      return 444;
    }
  }
}
```

Clients will get no HTTP response, just a closed connection.

```
$ curl -i http://localhost:8083/test_444

curl: (52) Empty reply from server
```

The Nginx server will log a 444 status code when this happens.

```
nginx-nginx-1  | 192.168.65.1 - - [12/Aug/2025:05:21:38 +0000] "GET /test_444 HTTP/1.1" 444 0 "-" "curl/8.7.1"
```

This confirms that no HTTP response was sent, just an immediate connection close.

## 499 Client Closed Request

499 Client Closed Request is defined in [ngx_http_request.h](https://github.com/nginx/nginx/blob/251444fcf4434bfddbe3394a568c51d4f7bd857f/src/http/ngx_http_request.h#L136) as:

```
/*
 * HTTP does not define the code for the case when a client closed
 * the connection while we are processing its request so we introduce
 * own code to log such situation when a client has closed the connection
 * before we even try to send the HTTP header to it
 */
#define NGX_HTTP_CLIENT_CLOSED_REQUEST     499
```

The point is that the status code is logged when a client has closed the connection before the nginx server even tries to send the HTTP header to it. Let's see examples.

You can test this behavior by setting up a nginx server as a proxy to a slow backend server like this nginx config (you can get the whole code at the appendix section of this article):

```
http {
  upstream slow_ruby {
    server localhost:8081;
  }

  // ...

  server {
    listen 8083;

    // ...
    location /slow-process {
      # Proxy to the slow Ruby service
      proxy_pass http://slow_ruby;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
      
      # Enable debug logging for proxy
      proxy_intercept_errors on;
      
      # Set reasonable timeouts
      proxy_connect_timeout 10s;
      proxy_send_timeout 60s;
      proxy_read_timeout 60s;
    }
  }
}
```

Then you can test this behavior by sending a request to the nginx server with a slow backend server like this:

```
$ curl -i -v http://localhost:8083/slow-process
# Cancel the request after 1 second
```

You will see a 499 status code in the nginx logs.

```
127.0.0.1 - - [14/Aug/2025:09:48:39 +0900] "GET /slow-process HTTP/1.1" 499 0 "-" "curl/8.7.1" "-"
```

This confirms that the nginx server logged a 499 status code when the client closed the connection before the nginx server even tried to send the HTTP header to it.

Specifically, [ngx_http_upstream](https://github.com/nginx/nginx/blob/1a82df8cca80458fc3da0968f64624f40cafdf37/src/http/ngx_http_upstream.c#L1418) module handles the 499 status code response.

```
static void
ngx_http_upstream_check_broken_connection(ngx_http_request_t *r,
    ngx_event_t *ev)
    {
        // ...

        if (!u->cacheable && u->peer.connection) {
            ngx_log_error(NGX_LOG_INFO, ev->log, ev->kq_errno,
                          "kevent() reported that client prematurely closed "
                          "connection, so upstream connection is closed too");
            ngx_http_upstream_finalize_request(r, u,
                                               NGX_HTTP_CLIENT_CLOSED_REQUEST);
            return;
        }
    }
```

There are some other functions that depends on `NGX_HTTP_CLIENT_CLOSED_REQUEST`, such as those for HTTP/2, HTTP/3, and so on. 

Let's see another example. This example is the case that the client sent a request to the nginx server endpoint that is slow due to the late limit.

```
http {
  // ...

  server {
    listen 8083;

    location /slow-body-response {
      # Simulate a slow endpoint by limiting response rate
      limit_rate 100;
      return 200 "This response will be slow due to rate limiting. Adding more content to make it take longer. This is a slower response for testing purposes. This should take more than 5 seconds to complete. Adding even more content to extend the response time further. The limit_rate directive controls how fast nginx sends the response to the client. With 1 byte per second, this response will be very slow indeed.\n";
    }
  }
}
```

Then you can test this behavior by sending a request to the nginx server with a slow body response like this:

```
$ curl -i -v http://localhost:8083/slow-body-response
# Cancel the request after 1 second
```

In this case, you will NOT see a 499 status code in the nginx logs, instead you will see a 200 status code.

```
127.0.0.1 - - [14/Aug/2025:09:52:56 +0900] "GET /slow-body-response HTTP/1.1" 200 51 "-" "curl/8.7.1" "-"
```

This is because the nginx server has already sent the HTTP header response to the client, so the client has closed the connection. The 499 status code is not logged in this case even if the client has closed the connection before the nginx server has send the complete response, including the body.

Interestingly, nginx server internally logs a 499 status code in a debug log depsite of the nginx server logs 200 in its access log as the same as the above example (e.g., `http finalize request: 499`).

```
2025/08/14 10:33:23 [debug] 83064#0: *2 HTTP/1.1 200 OK
Server: nginx/1.26.2
Date: Thu, 14 Aug 2025 01:33:23 GMT
Content-Type: text/plain
Content-Length: 400
Connection: keep-alive

2025/08/14 10:33:25 [info] 83064#0: *2 [src/http/ngx_http_request.c:3096] ngx_http_read_client_request_body_handler(): finalizing HTTP request with client closed request, client: 127.0.0.1, server: localhost, request: "GET /slow HTTP/1.1", host: "localhost:8084"
2025/08/14 10:33:25 [debug] 83064#0: *2 http finalize request: 499, "/slow?" a:1, c:1
```

This indicates that the nginx server recognizes the client has closed the connection before the nginx server has sent the complete response, but the nginx server logs 200 because 499 is only used when the connection closed before the server has sent a HTTP header response as the comment on the nginx source code says.

## Conclusion

Understanding HTTP status codes is fundamental for SREs, but Nginx's custom status codes add another layer of complexity that's crucial for effective troubleshooting and monitoring. While RFC 9110 defines the standard range of 100-599, Nginx extends this with proprietary codes like 444, 494-497, and 499 to handle specific scenarios that aren't covered by the standard.

The key takeaway is that Nginx-specific status codes serve distinct purposes:

- **444 No Response**: A security-focused code that immediately closes connections without sending HTTP responses, ideal for blocking malicious requests
- **494-497 SSL-related codes**: Handle various SSL/TLS certificate and protocol issues that standard HTTP codes can't adequately represent
- **499 Client Closed Request**: Logs when clients disconnect before receiving HTTP headers, providing visibility into connection stability issues

These custom codes are particularly valuable for SREs because they offer granular visibility into connection handling, security events, and client behavior patterns. When you see a 499 in your logs, you know the client disconnected early in the request lifecycle. A 444 indicates a deliberate security measure was triggered. This level of detail helps distinguish between server errors, client issues, and intentional security actions.

For production environments, monitoring these Nginx-specific status codes alongside standard HTTP responses gives you a complete picture of your application's health and security posture. They're not just logging artifacts—they're actionable signals that can help you identify connection issues, security threats, and performance bottlenecks before they impact your users.

Remember that while these codes aren't part of the HTTP standard, they're essential tools in the Nginx ecosystem for building robust, secure, and observable web applications.

## Appendix

### Nginx config for examples

You can get the whole code at [the github repository](https://github.com/hgsgtk/infra-experiments/tree/main/nginx). Here are the important pointsto note:

* docker-compose.yml

```
services:
  nginx:
    image: nginx:alpine
    ports:
      - "8083:8080"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - nginx_cache:/var/cache/nginx
    depends_on:
      - slow-ruby-service

  slow-ruby-service:
    build:
      context: ./upstream
      dockerfile: Dockerfile.ruby
    ports:
      - "8081:8081"
    environment:
      - RUBY_ENV=production

volumes:
  nginx_cache:
```

* nginx.conf

```
events {}
error_log stderr debug;

http {
  # Enable debug logging for various modules
  error_log stderr debug;
  
  # Define upstream for Ruby service
  upstream slow_ruby {
    server slow-ruby-service:8081;
  }

  # Define proxy cache
  proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m use_temp_path=off;

  # Define log format for access logs
  log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';

  # HTTP server
  server {
    listen 8080;
    server_name localhost;
    
    # Enable access logging to stdout
    access_log /dev/stdout main;

    location / {
      return 200 "Welcome to nginx server\n";
    }

    location /slow-process {
      # Proxy to the slow Ruby service
      proxy_pass http://slow_ruby;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
      
      # Enable debug logging for proxy
      proxy_intercept_errors on;
      
      # Set reasonable timeouts
      proxy_connect_timeout 10s;
      proxy_send_timeout 60s;
      proxy_read_timeout 60s;
    }

    location /cached-endpoint {
      # Enable proxy cache for this endpoint
      proxy_cache my_cache;
      proxy_cache_key "$scheme$request_method$host$request_uri";
      proxy_cache_valid 200 302 10m;  # Cache successful responses for 10 minutes
      proxy_cache_valid 404 1m;       # Cache 404s for 1 minute
      proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
      proxy_cache_lock on;
      proxy_cache_lock_timeout 5s;
      
      # Enable debug logging for cache
      proxy_cache_bypass $http_pragma;
      proxy_cache_revalidate on;
      
      # Add cache headers to response for debugging
      add_header X-Cache-Status $upstream_cache_status;
      add_header X-Cache-Key $request_uri;
      add_header X-Cache-Valid $upstream_cache_status;
            
      # Proxy to the slow Ruby service
      proxy_pass http://slow_ruby;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
      
      # Set reasonable timeouts
      proxy_connect_timeout 10s;
      proxy_send_timeout 60s;
      proxy_read_timeout 60s;
    }

    location /slow-body-response {
      # Simulate a slow endpoint by limiting response rate
      limit_rate 100;  # 100 bytes per second
      return 200 "This response will be slow due to rate limiting. Adding more content to make it take longer. This is a slower response for testing purposes. This should take more than 5 seconds to complete. Adding even more content to extend the response time further. The limit_rate directive controls how fast nginx sends the response to the client. With 1 byte per second, this response will be very slow indeed.\n";
    }

    location /test_444 {
      return 444;
    }
  }

}
```

* Dockerfile.ruby

```
FROM ruby:3.2-alpine

WORKDIR /app

# Copy the Ruby service
COPY slow-ruby-service.rb .

# Make the script executable
RUN chmod +x slow-ruby-service.rb

# Expose the port
EXPOSE 8081

# Run the service
CMD ["ruby", "slow-ruby-service.rb"]
```

* slow-ruby-service.rb

```
#!/usr/bin/env ruby

require 'socket'
require 'json'

class SlowRubyService
  def initialize(port = 8081)
    @port = port
    @server = TCPServer.new('0.0.0.0', @port)
    puts "Slow Ruby Service listening on port #{@port}"
  end

  def start
    loop do
      client = @server.accept
      Thread.new { handle_request(client) }
    end
  rescue Interrupt
    puts "\nShutting down Slow Ruby Service..."
    @server.close
  end

  private

  def handle_request(client)
    request_line = client.gets
    return unless request_line

    method, path, version = request_line.split(' ')
    
    # Read headers
    headers = {}
    while (line = client.gets.strip) && !line.empty?
      key, value = line.split(': ', 2)
      headers[key.downcase] = value if key && value
    end

    # Read body if present
    body = ""
    if headers['content-length']
      body = client.read(headers['content-length'].to_i)
    end

    puts "Received #{method} request to #{path}"
    puts "Headers: #{headers.inspect}"
    puts "Body: #{body}" unless body.empty?

    # Simulate slow processing
    sleep_time = case path
                 when '/slow-process'
                   5  # 5 seconds delay
                 when '/slow-api'
                   3  # 3 seconds delay
                 else
                   1  # 1 second default delay
                 end

    puts "Processing request for #{sleep_time} seconds..."
    sleep(sleep_time)

    # Send response
    response_body = {
      message: "Request processed successfully",
      path: path,
      method: method,
      processing_time: sleep_time,
      timestamp: Time.now.strftime("%Y-%m-%dT%H:%M:%S%z"),
      headers_received: headers.keys
    }.to_json

    response = [
      "HTTP/1.1 200 OK",
      "Content-Type: application/json",
      "Content-Length: #{response_body.bytesize}",
      # Uncomment when testing cache
      # "Cache-Control: no-cache, no-store, must-revalidate, max-age=0",
      # "Pragma: no-cache",
      # "Expires: 0",
      "",
      response_body
    ].join("\r\n")

    client.write(response)
    client.close
    puts "Response sent for #{path}"
  end
end

if __FILE__ == $0
  service = SlowRubyService.new
  service.start
end
```

### Build nginx with enabling debug logging from source 

You can build nginx with enabling debug logging from source. This is useful to see the internal behavior of nginx. The following procedure is tested on Apple M1 Pro (macOS 15.6).

1. Download the source code

Open Terminal and create a directory to work in:

```
mkdir -p ~/nginx-src && cd ~/nginx-src
```

Download the latest nginx source release:

```
curl -OL https://nginx.org/download/nginx-1.26.2.tar.gz
tar -xvzf nginx-1.26.2.tar.gz && rm nginx-1.26.2.tar.gz
```

Download PCRE (Perl Compatible Regular Expressions) library:

```
curl -OL https://github.com/PCRE2Project/pcre2/releases/download/pcre2-10.44/pcre2-10.44.tar.gz
tar -xvzf pcre2-10.44.tar.gz && rm pcre2-10.44.tar.gz
```

For SSL support, download OpenSSL:

```
curl -OL https://github.com/openssl/openssl/releases/download/openssl-3.4.0/openssl-3.4.0.tar.gz
tar -xvzf openssl-3.4.0.tar.gz && rm openssl-3.4.0.tar.gz
```

2. Configure nginx for Compilation

Go into the nginx source directory:

```
cd nginx-1.26.2
```

Configure nginx to use the libraries you just downloaded (including SSL support):

```
./configure --with-pcre=../pcre2-10.44/ --with-openssl=../openssl-3.4.0/ --with-http_ssl_module
```

3. Compile and Install nginx

Build and install nginx:

```
sudo make && sudo make install
```

The default installation path is /usr/local/nginx. You may change this with --prefix=<path> in the configure step if you wish.

Add nginx binary to your PATH for convenience:

```
export PATH="/usr/local/nginx/sbin:$PATH"
```

4. Start nginx

Start the nginx server:

```
sudo /usr/local/nginx/sbin/nginx -c /path/to/your/nginx.conf
```

Open http://localhost/ in your browser. You should see the Welcome to nginx! page.

5. Basic Management

To stop nginx:

```
sudo /usr/local/nginx/sbin/nginx -s stop
```

To reload configuration:

```
sudo /usr/local/nginx/sbin/nginx -s reload
```
