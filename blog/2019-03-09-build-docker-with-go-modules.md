Go1.11がリリースされたことにより、[Modules](https://github.com/golang/go/wiki/Modules)がサポートされました。
2019年3月9日時点では、Go1.12になってますね。
Modulesを利用する場合、それに伴いこれまでのDockerfileと微妙に異なる点についてです。

# Modulesとは
https://github.com/golang/go/wiki/Modules

ローカル環境での使い方

# Dockerfile

```Dockerfile
FROM golang:1.12.0-alpine3.9 as builder

RUN apk add --no-cache bash \
    ca-certificates \
    curl \
    git

ENV GO111MODULE=on

WORKDIR /go/src/github.com/hgsgtk/health-endpoint/src
COPY . /go/src/github.com/hgsgtk/health-endpoint/src

# module cache
COPY go.mod .
COPY go.sum .

RUN go mod download

RUN go build -o server

FROM alpine

COPY --from=builder /go/src/github.com/hgsgtk/health-endpoint/src/server .

ENTRYPOINT ["./server"]
```

GOPATHの設定が不要

go.mod / go.sumをキャッシュ
go mod download

## レイヤーキャッシュ


# refs
- [Container Solution Blog: aster builds in Docker with Go 1.11](https://container-solutions.com/faster-builds-in-docker-with-go-1-11/)
