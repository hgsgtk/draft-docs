# TL;DR

- PSR-3 Logger Interfaceには、``Psr\Log\NullLogger``という実装が提供されている
- ``Psr\Log\NullLogger``は、オブジェクト指向言語のデザインパターンとして一般的に知られるNullObjectパターンと評価できる
- CakePHPの内部実装と使用方法からそのユースケースを眺める

# 背景

CakePHP 4.0 の変更点を追っている際に、次のような記述があった。

> Cake\Database\Connection::setLogger() no longer accepts null to disable logging. Instead pass an instance of Psr\Log\NullLogger to disable logging.

https://book.cakephp.org/4/en/appendices/4-0-migration-guide.html

これまで、``null``を渡してロギングを無効にする方法をとっていたが、``Psr\Log\NullLogger``を渡して無効化するように案内されていた。

``NullXxx``と見ると、NullObjectパターンが想起されたので、``Psr\Log\NullLogger``を入り口として、**PSR-3**と**Null Object Patttern**の説明を試みる。

# PSR

そもそも、PSRとは、**PHP Standards Recommendations**[^1]の略で、PHP-FIG[^2]というコミュニティ団体による、コミュニティにおいて（ある程度）影響が大きい標準化勧告である。PHP開発に馴染みのある方であれば、普段良く使用しているフレームワークやライブラリが、この勧告によって提示されているInterfaceをサポートしていたりする。AcceptされたものからDraft段階のものを含めて**No.19**まで提案されており、冒頭に上げたCakePHPもPSR-15やPSR-16などをサポートしているフレームワークの一つです。

[^1]: PHP Standards Recommendations (https://www.php-fig.org/psr/)

[^2]: PHP-FIG (https://www.php-fig.org/)

# PSR-3 Logger Interface

その中の、No.3であるPSR-3[^3]は、すでにAcceptされているPSRで、**Logger Interface**を規定する。つまり、ロガーライブラリの標準となる共通インターフェースを定めようというものになる。

[https://www.php-fig.org/psr/psr-3/:embed:cite]

[^3]: PSR-3 (https://www.php-fig.org/psr/psr-3/)

>  The main goal is to allow libraries to receive a Psr\Log\LoggerInterface object and write logs to it in a simple and universal way.

基本的なインターフェースとして、RFC5425[^4]で定義された**8つのレベル**にログを書き込むためのメソッドを公開する。そして、9つめのメソッドとして、ログレベルを第一引数として受け入れる``log``を公開する。

[^4]: RFC5425 (https://tools.ietf.org/html/rfc5424)

```php
<?php
namespace Psr\Log;

interface LoggerInterface
{
    public function emergency($message, array $context = array());
    public function alert($message, array $context = array());
    public function critical($message, array $context = array());
    public function error($message, array $context = array());
    public function warning($message, array $context = array());
    public function notice($message, array $context = array());
    public function info($message, array $context = array());
    public function debug($message, array $context = array());
    /**
     * @throws \Psr\Log\InvalidArgumentException
     */
    public function log($level, $message, array $context = array());
}
```

https://github.com/php-fig/log/blob/4aa23b5e211a712047847857e5a3c330a263feea/Psr/Log/LoggerInterface.php#L3

``log``メソッドのPHPDoc[^5]を見ると明らかですが、ログレベルがRFC5425[^4]で規定されたレベル以外の場合は、``\Psr\Log\InvalidArgumentException``を返すことが指定されている。

[^5]: phpDocumentor (https://www.phpdoc.org/)

# Psr\Log\NullLogger

この、PSR-3[^3]のLogger Interfaceの「1.4 Helper classes and interfaces[^5]」にて、``Psr\Log\NullLogger``がInterfaceと一緒に提供されている点について説明されている。

> The Psr\Log\NullLogger is provided together with the interface. It MAY be used by users of the interface to provide a fall-back “black hole” implementation if no logger is given to them. However, conditional logging may be a better approach if context data creation is expensive.

[^5]: PSR-3 1.4 Helper classes and interfaces (https://www.php-fig.org/psr/psr-3/#14-helper-classes-and-interfaces)

フォールバックの実装を提供するために使用することができる*black hole*実装となっています。

```php
<?php

namespace Psr\Log;

/**
 * This Logger can be used to avoid conditional log calls.
 *
 * Logging should always be optional, and if no logger is provided to your
 * library creating a NullLogger instance to have something to throw logs at
 * is a good way to avoid littering your code with `if ($this->logger) { }`
 * blocks.
 */
class NullLogger extends AbstractLogger
{
    /**
     * Logs with an arbitrary level.
     *
     * @param mixed  $level
     * @param string $message
     * @param array  $context
     *
     * @return void
     *
     * @throws \Psr\Log\InvalidArgumentException
     */
    public function log($level, $message, array $context = array())
    {
        // noop
    }
}
```

[https://github.com/php-fig/log/blob/4aa23b5e211a712047847857e5a3c330a263feea/Psr/Log/NullLogger.php#L13:embed:cite]

継承している``Psr\Log\AbstractLogger``にて、8つのログレベルの公開メソッドが実装されているため、LoggerInterfaceを満たすものとなっています。

## フレームワークにおける使用例

この``Psr\Log\NullLogger``の使用例をCakePHPの内部実装で見てみます。PSR-3の**\Psr\Log\LoggerInterface**への依存へ変更しているPRから、見ていきます。

```php
-    public function setLogger(?QueryLogger $logger)
+    public function setLogger(LoggerInterface $logger)
```

[https://github.com/cakephp/cakephp/pull/13276/files#r286279848:embed:cite]

CakePHP3.x以前では、ロギングを無効にするために、``Cake\Database\Connection::setLogger()``に``null``を渡す方法をとっていました。

```php
        $connection = ConnectionManager::get('test');
        $connection->setLogger(null); // 無効
```

 この場合、以降ロガーを呼び出す際に、``if ($this->_logger === null)``といった条件分岐などを使用してチェックするようなコードを用意することになります。

それに対して、NullLoggerを利用する場合は、そのようなチェックをせずとも、同一のインターフェースを利用することができます。

```php
        $connection = ConnectionManager::get('test');
        $connection->setLogger(new NullLogger()); // 無効
```

このようなメリットを享受できるコード設計パターンは、**Null Object Pattern**というパターンの特徴として見ることができます。

# Null Object Pattern

Null Object Pattern[^6]とは、**オブジェクト指向言語に**おけるパターンで、**参照される値がないか、定義されたニュートラル（``null``）動作を持つオブジェクト**を指します。「プログラムデザインのためのパターン言語―Pattern Languages of Program Design選集[^7]」や、「リファクタリング 既存のコードを安全に改善する（第2版）[^8]」にて、本として初めて紹介されたパターンとなります。（※ 「リファクタリング 既存のコードを安全に改善する（第2版）[^8]」では、**Special Case Pattern**とも表現されています。）

[^6]: Null Object Pattern (https://en.m.wikipedia.org/wiki/Null_object_pattern)

[^7]: プログラムデザインのためのパターン言語―Pattern Languages of Program Design選集 (https://www.amazon.co.jp/dp/4797314397)

[^8]: リファクタリング 既存のコードを安全に改善する（第2版） (https://www.amazon.co.jp/dp/B0827R4BDW)

これは、GoF (Gang of Four)の23個のデザイン・パターンではありませんが、現代のプログラミングの現場において広く知られる「デザイン・パターン」といえます。

先程のCakePHPの例では、次のようなクラス関係を表現しています。

[f:id:khigashigashi:20200102183151p:plain]

デザイン・パターンは、オブジェクト指向ソフトウェアを設計する際の経験を記録、カタログ化したものです。繰り返し現れる構造をパターンとしてまとめたものです。「問題」・「解決」・「コンテクスト」の大きく3つの要素をそのパターンから見出すことができます。

# まとめ

PSR-3の標準的なインターフェースとともに提供されているクラスから、NullObjectというパターンを紹介しました。
