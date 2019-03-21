# TL;DR
- 自動ユニットテストが何を目指すべきかを『[xUnit Test Patterns: Refactoring Test Code](https://www.amazon.co.jp/dp/0131495054/ref=cm_sw_r_tw_dp_U_x_Y8kJCb6EX02F6)』から抑える
- テストの自動化の目的は、「コスト削減」・「品質の向上」・「コードへの理解の向上」

# 『xUnit Test Patterns』 とは
『[xUnit Test Patterns: Refactoring Test Code](https://www.amazon.co.jp/dp/0131495054/ref=cm_sw_r_tw_dp_U_x_Y8kJCb6EX02F6)』とは、2007年に発売された書籍です。自動ユニットテストにおける原則・パターンなどが体系的にまとめられています。

今回のエントリでは、『xUnit Test Patterns: Refactoring Test Code』の内容をベースとしています。
具体的には、「Chapter3 Goals of Test Automation」をベースとして、関連書籍・実践の感覚値を踏まえてまとめています。

# よいテストを書く難しさ
よいテストを書くことは難しく、理解するのが難しいテストコードをメンテナンスするのはさらに難しいです。要因として、テストコードはソフトウェアを利用する顧客が支払うものではない **Optional** である点があります。テストの維持が難しく高コストになると、それらを捨てたくなる強い誘惑があります。 *keep the bar green to keep the code clean* の原則を一度諦めると、自動化テストの価値は失われてしまいます。

## 補足：現場での難しさ
私自身もこの難しさは所属企業で実感したものです。具体的には、ユニットテストがない状態からテストカバレッジをあげていくというフェーズにおいてです。ユニットテストを覚える学習コストや、不慣れなゆえに発生した「壊れやすいテスト([Fragile Test](http://xunitpatterns.com/Fragile%20Test.html))」に対するトラブルは、ユニットテストを「障害物」ととらえてしまう状況になりえました。これらの課題と対策については、[PHPバージョンアップと決済リプレイスを支えたユニットテスト at PHPカンファレンス2018](https://speakerdeck.com/hgsgtk/phpbaziyonatuputojue-ji-ripureisuwozhi-etayunitutotesuto-number-phpcon)という資料にて詳述していますので、そちらも合わせてご参照ください。
以降では、この難しさに対してどう立ち向かっていくかという話を進んでいきます。

# テストの経済性 （Economics of Test Automation）
自動化テストを構築・メンテナンスするのには、それ自身にコストがかかります。 しかし、自動化テストの構築・メンテナンスにかかる追加コストは、テストによる節約コストによって **相殺** されます。テストによる節約コストとは、 **手動によるユニットテストの回避** や **デバッグ/トラブルシューティングの削減** ・ **正式なテストフェーズ・本番稼働の初期まで検出されなかった欠陥の修正コスト** です。
テスト自身のコストとプロダクト自体の開発コストは次の図によって表せます。

画像： img/Economics-of-Test-Automation.png。

最初は、自動化ユニットテストという新しい技術に対する学習コストとそれに対する実践コストとして、コストの嵩む時期が訪れます。しかし、じきに追加コストが落ち着いて来るとともに、テストによる節約コストと相殺されていきます。

しかし、テストの読み書きが難しかったり頻度高くコストの掛かるメンテナンスを要すような状況では、次の図の示すようにソフトウェア開発におけるトータルのコストは上がっていきます。

画像： img/Economics-of-Test-Automation-anti.png。

後者の状況にならないためにも、本書で語られているような **ユニットテストの目指すべきところ** を抑えた上で、プロジェクト全体のコストを削減する「自動ユニットテスト環境」を目指していきましょう。

# テスト自動化のゴール
ここで、本エントリのタイトルにもある「ユニットテストの目的」と大きく関係する「テスト自動化のゴール」についてです。『xUnit Test Patterns: Refactoring Test Code』では、6つの目的をあげています。

1. Tests should help us improve quality.
2. Tests should help us understand the SUT.
3. Tests should reduce (and not introduce) risk.
4. Tests should be easy to run.
5. Tests should be easy to write and maintain.
6. Tests should require minimam maintenance as the system evolves around them.

最初の3つの目的は、 **テストがもたらす価値** について示しています。一方、以降の3つは **テスト自身の特徴** について注目したものです。

## 用語定義：SUT
[xUnit Test Patterns: Refactoring Test Code](https://www.amazon.co.jp/dp/0131495054/ref=cm_sw_r_tw_dp_U_x_Y8kJCb6EX02F6)では、 **SUT** という言葉が頻繁に登場します。前提としてこの言葉を抑えます。

[SUT](http://xunitpatterns.com/SUT.html)は、 **system under test** の略語です。これは、「テストしている対象」を示すものです。今回はユニットテストですので、テストスクリプトが実行する「テスト対象のクラスやメソッド」のことを指します。

SUTという言葉を抑えたところでさっそくテスト自動化のゴールを見ていきましょう。

## Tests should help us improve quality.（品質向上）
テストをすることで代表的な理由は、品質保証（Quality Assurance）です。

### Tests as Specification （"仕様"としてのテスト）
テスト駆動開発（Test-Driven Development, TDD）やテストファーストを行った場合、SUT作成前に、テストによって **SUTが何をするべきなのか** を把握できます。「正しいソフトウェアを構築している」ことを確実とするために、テストには **「SUTがどのように使われるか」** を反映させなければなりません。「SUTがどのように使われるか」をテストに反映させようとすることで、あいまいな要求や自己矛盾に開発者自身が気付けます。それによって、**"仕様"（Specification）の質** が向上し、結果として **ソフトウェアの品質向上** につながります。

### Bug Repellent（バグよけ）
テストによってバグが見つかります。しかし、自動テストの目的はバグを見つけることではありません。
自動テストは **新たにバグが入り込むことを防ぐ** ことを目的としています。実行される回帰テスト（*Regression Test*）がバグをピンポイントで指摘するため、バグが入りこなくなります。

### Defect Localization
それぞれのユニットテストが十分に小さい場合、テストの失敗結果によってバグをすばやく特定できます。顧客テスト（受け入れテスト）では、 **動作がおかしい** ことを検知する一方、ユニットテストは **なぜおかしい** のかを伝えてくれます。この利点を本書では *Defect Localization* と呼んでいます。

また、「顧客テストは失敗しているがユニットテストは成功している」という状況を、[Missing Unit Test](http://xunitpatterns.com/Production%20Bugs.html#Missing%20Unit%20Test)と呼びます。*Defect Localization* はすばらしい利点です。しかし、考えうるすべてのシナリオに対してユニットテストを書かないと達成できません。

## Tests should help us understand the SUT.（SUTの理解促進）
テストは、テストの読み手に対して **「SUTがどのように機能するか」** を示せます。
この特性は、 **Tests as Documentation**（"ドキュメント"としてのテスト）と呼ばれます。自動化テストがある場合、テストを *"ドキュメント"* として簡単に使用できます。
また、「**SUTが何をしているのか**」を知りたければ、Debugger（ex. [xdebug](https://xdebug.org/)）を起動してテストを実行し、シングルステップでコードがどのように動いているか見ることで、それを知ることができます。

## Tests should reduce (and not introduce) risk.（リスクを減らす）
### Tests as Safety Net（セーフティネット）
回帰テストが存在しない、いわゆる **レガシーコード** と立ち向かうのは憂鬱ですよね。コードの変更によって、「何か壊していないか」を知ることができないのはリスクです。結果的に、開発速度は遅く慎重なものになり、変更時の手動での大量の検査をするといったことになります。

一方、回帰テストが存在する場合、非常にすばやくそれらを実行できます。「何か壊していないか」は **テストの失敗によって** 知ることができます。
しかし、すでに紹介した [Missing Unit Test](http://xunitpatterns.com/Production%20Bugs.html#Missing%20Unit%20Test) があれば、セーフティネットに穴が空いている状態です。

### Do No Harm（害をもたらさない）
テストによってSUTに新たなリスクを生み出さないように注意しなければなりません。
まず守るべきものとして、**Keep Test Logic Out of Producton Code** 原則があります。これは、 **テストロジックをプロダクションコードから除外する** という内容で、たとえばプロダクションコード内に `if ($mode === 'test') {}`といったテストをするための *hook* を入れるのは避けるべきだというものです。

他のリスクは、「信頼していたコードが実は動かない」というケースです。よくある間違いは、 [Test Double](http://xunitpatterns.com/Test%20Double.html)を過剰に使用してしまうことです。このことは、もうひとつの重要な原則 **Don't Modify the SUT** をもたらしています。

ここで登場した２つの原則については、次のURLにてまとめましたので合わせてご参照ください。

[xUnit Test Patternsから学ぶ12個のユニットテストの原則 on Qiita](https://qiita.com/hgsgtk/items/a3186a250d36d3b224d9)

## Tests should be easy to run.（実行が簡単）
「ワンクリックで簡単に実行できる」ことについてです。テストの実行を容易とするために、次の4つのゴールがあります。

- Fully Automated Tests
- Self-Checking Tests
- Repeatable Tests
- *Independent Test（独立したテスト）*

これらの多くは、[JUnit](https://junit.org/junit5/)や[PHPUnit](https://phpunit.de/)といったテスティングフレームワークを使用していると、自然に達成しているゴールです。しかし、意識しないとその利点を意図せず壊してしまう可能性がある部分もあるので、おさえていきましょう。

*Fully Automated Tests* は、「テストは **手動介入** なしで実行できなければいけない」ということです。そして、 *Self-Checking Tests* は、「テスト自身がテストの失敗を報告する」という特性を表しています。
実際、テスティングフレームワークを使用している場合、用意されているコマンドを実行して、出力された結果を確認します。普段のテスト体験について思い出すとこの２つは想像しやすいですね。

### Repeatable Test（繰り返し実行可能）
*Repeatable* なテストでは、何度実行しても同じ結果が得られます。実行のたびに結果が変わりうるテストは、[Erratic Test](http://xunitpatterns.com/Erratic%20Test.html)（不安定なテスト）のひとつです。
インメモリなデータや、ローカル変数・フィールドのみを使用するテストなどが、*Repeatable* なテストの書き方としてあげられます。

## Tests should be easy to write and maintain.（書きやすく、メンテナンスしやすい）
プロダクションコードのコーディングは、頭の中の多数の情報を保持する必要がある難しい作業です。しかし、テストの場合は **「テストを書くこと」** ではなく **「テストすること」** に対して重点を置くべきです。そのため、テストコードは読み書きする上で **シンプル** でなければなりません。

### Simple Test（シンプルなテスト）
テストは **小さく** 、 **１回につきひとつのこと** をテストするべきです。複数機能を検証しているような場合は、それぞれ別々のテストメソッドに分割するなど、 **Verify One Confition per Test**（テストごとにひとつの条件を検証する）という原則を満たすテストコードにしていくべきです。

### Expressive Tests（表現力豊かなテスト）
[Test Utility Methods](http://xunitpatterns.com/Test%20Utility%20Method.html)を活用することで、「何をテストしたいのか」が伝わりやすくなります。

これの活用時のひとつは、DRY（Don't Repeat Yourself）原則がテストコードに対しても適用しうるときです。

しかし忘れてはいけないことは、テストコードは読み手に **意図を伝える** 必要があるという点です。それぞれのテストメソッドにて意図を伝えるためにコアとなるテストロジックは、それぞれの中にとどめておくのがいいでしょう。

### Separation of Concerns（関心の分離）
この文脈での「*Separation of Concerns*」は２つの側面があります。

1. **テストコードはプロダクションコードから分離する**
2. **それぞれのテストはひとつの関心に集中する**

このゴールが達成できていない例として、UIとビジネスロジックを同じ場所でテストすることが挙げられます。なぜなら、ひとつのテストで複数のことに対して関心を持ってしまうからです。これをやってしまうと、もしどれかひとつが失敗した場合にすべてが失敗することになります。

このように、「ひとつの関心に集中する」テストを書くには、ロジックをいくつか異なるコンポーネントに分割する必要が出てきます。結果として **Design for Testability（テスト容易な設計）** につながっていきます。

## Tests should require minimam maintenance as the system evolves around them.（システム変更に対して最小限のメンテナンス）


# Next
『[xUnit Test Patterns: Refactoring Test Code](https://www.amazon.co.jp/dp/0131495054/ref=cm_sw_r_tw_dp_U_x_Y8kJCb6EX02F6)』では、 **自動ユニットテストの原則** ・ **自動ユニットテストにおけるアンチパターン** についても言及されています。
こちらについても次のURLにてまとめましたので、合わせてご参照ください。

- [xUnit Test Patternsから学ぶ12個のユニットテストの原則 on Qiita](https://qiita.com/hgsgtk/items/a3186a250d36d3b224d9)
- [xUnit Test Patternsから学ぶテストアンチパターン on Speaker Deck](https://speakerdeck.com/hgsgtk/testing-anti-pattern-learned-in-xunit-test-pattern)

# References
## Books
- [xUnit Test Patterns: Refactoring Test Code](https://www.amazon.co.jp/dp/0131495054/ref=cm_sw_r_tw_dp_U_x_Y8kJCb6EX02F6)

## Articles
- [Xunit Test Patterns](http://xunitpatterns.com/)
  - [Fragile Test](http://xunitpatterns.com/Fragile%20Test.html)
  - [Missing Unit Test](http://xunitpatterns.com/Production%20Bugs.html#Missing%20Unit%20Test)
  - [Test Double](http://xunitpatterns.com/Test%20Double.html)
  - [Erratic Test](http://xunitpatterns.com/Erratic%20Test.html)
- [社内LT20170810(xUnit Test Patterns Chapter3について)](https://speakerdeck.com/o0h/she-nei-lt20170810) by [@o0h_](https://twitter.com/o0h_)
- [xUnit Test Patternsから学ぶ12個のユニットテストの原則 on Qiita](https://qiita.com/hgsgtk/items/a3186a250d36d3b224d9) by [@hgsgtk](https://twitter.com/hgsgtk)
- [xUnit Test Patternsから学ぶテストアンチパターン on Speaker Deck](https://speakerdeck.com/hgsgtk/testing-anti-pattern-learned-in-xunit-test-pattern) by [@hgsgtk](https://twitter.com/hgsgtk)
- [PHPバージョンアップと決済リプレイスを支えたユニットテスト #phpcon](https://speakerdeck.com/hgsgtk/phpbaziyonatuputojue-ji-ripureisuwozhi-etayunitutotesuto-number-phpcon) by [@hgsgtk](https://twitter.com/hgsgtk)
