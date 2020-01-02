# PSR-3のLoggerAwareInterfaceはどのように使われているのか

## TL;DR

TBD

## 背景

[PSR-3とCakePHPから見るNull Object Pattern][^1]という記事にて、PSR-3 Logger Interfaceについて触れました。

https://khigashigashi.hatenablog.com/entry/2020/01/02/183211

[^1]: PSR-3とCakePHPから見るNull Object Pattern (https://khigashigashi.hatenablog.com/entry/2020/01/02/183211)

公開時に[@wand_ta][wanda_ta]さんが、クラス図についてのありがたい指摘を頂いた後に、次のようなコメントをしておりました。

[wanda_ta]: https://twitter.com/wand_ta

<blockquote class="twitter-tweet" data-partner="tweetdeck"><p lang="ja" dir="ltr">LoggerAwareInterface@setLoggerっていつ使うんだろうって思ってたけど、setLoggerなFWが既に存在していたのを意識したのかな</p>&mdash; こべに(D. Horiyama) (@wand_ta) <a href="https://twitter.com/wand_ta/status/1212675588571271169?ref_src=twsrc%5Etfw">January 2, 2020</a></blockquote>

この、``LoggerAwareInterface``は、PSR-3の中で定義されているInterfaceなのですが、これについて、他のフレームワークはどういう風に使っているのかも気になったので、ここに調べました。

## PSR-3 Logger Interface

そもそも、PSRとは、**PHP Standards Recommendations**[^1]の略で、PHP-FIG[^2]というコミュニティ団体による、コミュニティにおいて（ある程度）影響が大きい標準化勧告です。

その中の、No.3であるPSR-3[^3]は、すでにAcceptされているPSRで、**Logger Interface**を規定しています。つまり、ロガーライブラリの標準となる共通インターフェースを定めようというものです。

## LoggerAwareInterface

今回の題材になっている``LoggerAwareInterface``は、下記のURLにて次のように説明されています。

https://www.php-fig.org/psr/psr-3/#14-helper-classes-and-interfaces

> The Psr\Log\LoggerAwareInterface only contains a setLogger(LoggerInterface $logger) method and can be used by frameworks to auto-wire arbitrary instances with a logger.

つまり、``setLogger(LoggerInterface $logger)``メソッドのみを持つInterfaceです。フレームワークがロガーと任意のインスタンスを自動接続するために使用できるとしています。

```php
<?php

namespace Psr\Log;

/**
 * Describes a logger-aware instance.
 */
interface LoggerAwareInterface
{
    /**
     * Sets a logger instance on the object.
     *
     * @param LoggerInterface $logger
     *
     * @return void
     */
    public function setLogger(LoggerInterface $logger);
}
```

https://github.com/php-fig/log/blob/f960c17b71ec7067eaa372f4712219a3c20e5926/Psr/Log/LoggerAwareInterface.php#L3

そして、任意のクラスでこのインターフェースを簡単に実装できるように、``LoggerAwareTrait``というトレイトが用意されています。

> The Psr\Log\LoggerAwareTrait trait can be used to implement the equivalent interface easily in any class. It gives you access to $this->logger.

```php
<?php

namespace Psr\Log;

/**
 * Basic Implementation of LoggerAwareInterface.
 */
trait LoggerAwareTrait
{
    /**
     * The logger instance.
     *
     * @var LoggerInterface
     */
    protected $logger;

    /**
     * Sets a logger.
     *
     * @param LoggerInterface $logger
     */
    public function setLogger(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }
}
```

https://github.com/php-fig/log/blob/4ee887bedd041aa86be43a7de62a13d9dd047a92/Psr/Log/LoggerAwareTrait.php#L8

インターフェースと実装は非常にシンプルですが、実際にはどのように使われているのでしょうか。いくつか、フレームワークを眺めてみたいと思います。

## CakePHPにおける使用例

## Laravelにおける使用例

## 定義の経緯

[@wand_ta][wanda_ta]さんが、解明してくださりました。

<blockquote class="twitter-tweet" data-partner="tweetdeck"><p lang="ja" dir="ltr"><a href="https://t.co/iNmJiM5j0Z">https://t.co/iNmJiM5j0Z</a><br>PSR-FIGのメーリングリストに入って漁ってみました<br>Symfonyにならってsetter-injection用に生やしたよう？ <a href="https://t.co/sgsCDebz2u">pic.twitter.com/sgsCDebz2u</a></p>&mdash; こべに(D. Horiyama) (@wand_ta) <a href="https://twitter.com/wand_ta/status/1212687417137758208?ref_src=twsrc%5Etfw">January 2, 2020</a></blockquote>
