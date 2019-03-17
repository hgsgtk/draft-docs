# TL;DR
- 自動ユニットテストの原則を『xUnit Test Patterns: Refactoring Test Code』からおさえる
- 関連書籍や実践の感覚値を随時補足していく

# xUnit Test Patterns
本エントリは、[xUnit Test Patterns: Refactoring Test Code](https://www.amazon.co.jp/dp/0131495054/ref=cm_sw_r_tw_dp_U_x_Y8kJCb6EX02F6)という書籍の「Chapter5 Principles of Test Automation」の内容をベースに、12個のユニットテスト原則についてまとめていきます。この書籍は、2007年に販売されたものですが、今でも十分役に立つユニットテストに関する原則を伝えています。

また、次のURLでも内容を見ることができます。

http://xunitpatterns.com/

# 自動ユニットテストの原則
ここで紹介されるものは、ユニットテストで確認したい quality のリストです。ですので、直接適用する「パターン」ではありません。
「何をやるか」よりも「なぜやるのか」という観点においてまとめられています。
本エントリでは、[xUnit Test Patterns: Refactoring Test Code](https://www.amazon.co.jp/dp/0131495054/ref=cm_sw_r_tw_dp_U_x_Y8kJCb6EX02F6)で紹介されている12個の原則をベースに、ほか関連書籍なども踏まえつつ「ユニットテストの原則」をおさえていきます。

## The Principles by xUnit Test Patterns
これは、次の12個の原則から構成されています。

1. Write the Tests First
2. Design for Testability
3. Use the Front Door First
4. Communicate Intent
5. Don't Modify the SUT
6. Keep Tests Independent
7. Isolate the SUT
8. Minimize Test Overlap
9. Minimize Untestable Code
10. Keep Test Logic Out of Production
11. Verify One Condition per Test
12. Test Concerns Separately

今回は、1〜4の内容を見ていきます。

## Write the Tests First （テストを最初に書く）
*Test-Driven Development(TDD)* あるいは *Test-First Development* として知られる原則です。TDDを支持する理由として主に２つあります。

### デバッグ作業の手間が省ける
ユニットテストを書くことによってデバッグ作業の手間が省けます。たとえば、ひとつのクラス・関数をデバッグする際に実行できるユニットテストがない場合、手動での実行などになります。ユニットテストを用意することによってデバッグ対象を単体で実行できるので、デバッグ作業の手間は大きく下がります。

### テスト容易性が強制される
コードを書く前にテストを書くことによって、テスト容易な設計が自然と強制されます。なぜなら、テストを書くには、テスト容易な設計である必要があるからです。
コードの設計とテスト容易な設計という観点を別観点として分けて考える必要がなくなります。

#### 補足：テストファーストと「よい設計」
ただし現実的には、テストを書いたからといって「よい設計」が生まれるわけではなく、テストをするための「多少の再利用性」が得られるという効用です。
[オブジェクト指向設計実践ガイド ～Rubyでわかる 進化しつづける柔軟なアプリケーションの育て方](https://gihyo.jp/book/2016/978-4-7741-8361-9)という書籍の「第9章 費用対効果の高いテストを設計する」では、「*初級の設計者はテストファーストでコードを書くことが最も有益です。*」と書かれています。
訓練として、テストを最初に書くアプローチを実践して習得することは最低限の成果を実現するために非常に有用といえます。

そのうえで、Test Drive Development の手法に熟練していくことで、よい設計を導いていくアプローチを実現できます。[テスト駆動開発](https://www.amazon.co.jp/dp/4274217884/ref=cm_sw_r_tw_dp_U_x_c-CJCb9GKFXC4)や[実践テスト駆動開発 (Object Oriented SELECTION) ](https://www.amazon.co.jp/dp/4798124583/ref=cm_sw_r_tw_dp_U_x_c.CJCb2G9QMZF)といった書籍が参考になります。

## Design for Testability （テスト容易性を設計する）
「Write the Tests First」をしない選択をしなかったなど、テスト容易性が設計されていないコードに対してよりこの原則が重要です。レガシーソフトウェアに対するユニットテストの難しさは、この設計がされていないことによって引き起こります。

この課題に対して、Michael Feathers氏が、[Working Effectively with Legacy Code](https://www.amazon.com/Working-Effectively-Legacy-Michael-Feathers/dp/0131177052)や[レガシーコード改善ガイド](https://www.shoeisha.co.jp/book/detail/9784798116839)といった書籍でテクニックを紹介しています。

## Use the Front Door First （最初に正面玄関を使う）
*Front Door First* とも呼ばれます。オブジェクトは、外部から利用することを期待される *public* インタフェースや、内部からのみ利用することを期待される *private* インタフェースがあります。
テストによって使用するインタフェースは、それぞれテストの堅牢性に影響を与えます。
フィクスチャを設定したり、予想される結果やテストを検証するために *バックドア操作* を使用すると、テストのメンテナンスを頻繁に行う必要がある状態になります。それは、壊れやすいテスト（*Fragile Test*）」のひとつ *Overcoupled Software* と呼ばれています。
また、振る舞い検証およびモックオブジェクトの過剰使用は、これまた壊れやすいテスト（*Fragile Test*）のひとつ *Overspecified Software* と呼ばれる状況になります。それによって、テストがより脆弱になり、開発者のリファクタリングを妨げる可能性があります。

すべての選択肢が等しく有効であれば、[round trip test](http://xunitpatterns.com/round%20trip%20test.html) を用いるべきです。[round trip test](http://xunitpatterns.com/round%20trip%20test.html) とは、 *front door(public interface)* のみを介してテスト対象システム（[SUT](http://xunitpatterns.com/SUT.html)）と対話するテストです。 *public* インタフェースを通してオブジェクトをテストし、それが正しく振る舞っているかを確認するための状態検証を行います。

[round trip test](http://xunitpatterns.com/round%20trip%20test.html)が期待する振る舞いを正確に記述するのに十分でない場合、[layer-crossing test](http://xunitpatterns.com/layer-crossing%20test.html)などのテスト方法を活用できます。

### 補足：privateインタフェースをテストするか
実際、 *public* インタフェースを用いることを優先することは設計の指標としても有用だと考えます。 *public* インタフェースは外から使用することを期待するため、 *private* メソッドとの比較すると、変更の頻度が少ない安定したインタフェースと考えられます。逆を返すと、 *private* インタフェースは不安定なインタフェースとなるため、テストのメンテナンス頻度が高いテストを生み出すことになります。壊れやすいテストのひとつ *Overcoupled Software* を生み出さないためには、「なるべく」 *private* インタフェースをテストする必要がない設計を考えるのがよいでしょう。

## Communicate Intent （意図を伝える）
*Higher-Level Language, Single-Glance Readable* とも知られる原則です。自動化されたテストはプログラムであるため、対象をテストするために、必要な詳細ロジックをすべて実装することが重要です。
しかし、テストをメンテナンスする開発者が、 **理解しやすくメンテナンスしやすい** テストにすることも重要です。

大量のコードが含まれていたり、[Conditional Test Logic](http://xunitpatterns.com/Conditional%20Test%20Logic.html)は、*曖昧なテスト*([Obscure Tests](http://xunitpatterns.com/Obscure%20Test.html))と呼ばれ、理解するのが難しいものになります。

[Test Utility Methods](http://xunitpatterns.com/Test%20Utility%20Method.html)のライブラリを使用すると、詳細をすべてコーディングする必要がないため、テストの作成コストが下がります。また、Test Utility Methodにまとめることでテストケースの意図が伝わりやすいものになります。

## Don't Modify the SUT（SUTを修正しない）

Test Hooks
Test Double

## Keep Tests Independent（テストを独立させる）
手動テストでは
自動テストでは

Fresh Fixture
Shared Fixture

## Isolate the SUT（SUTを隔離する）
もし、テスト対象のSUTが他のソフトウェアに依存している場合、他のソフトウェアの動作の変更によってテストが突然失敗することがあります。たとえば、SUTが外部システムに依存しているようなケースです。
これは、 *Context Sensitivity* と呼ばれる壊れやすいテストのひとつです。

これを避けるために、テストを完全に制御しながら依存関係の可能性のあるすべての反応をソフトウェアに注入できる必要があります。具体的には、依存関係のあるソフトウェアを[Dependency Injection](http://xunitpatterns.com/Dependency%20Injection.html)や[Dependency Lookup](http://xunitpatterns.com/Dependency%20Lookup.html)、[Test-specified Subclass](http://xunitpatterns.com/Test-Specific%20Subclass.html)を用いた上書きによって、[Test Double](http://xunitpatterns.com/Test%20Double.html)に置き換える方法によって制御できます。

## Minimize Test Overlap（テストの重複を最小限に抑える）

## Minimize Untestable Code（テスト不可能なコードを最小限に抑える）

## Keep Test Logic Out of Production Code（テストロジックをプロダクションコードから除外する）

## Verify One Condition per Test（テストごとに一つの条件を検証する）

## Test Concerns Separately（）

## Ensure Commensurate Effort and Responsibility（）

# Refs
## 参考書籍
- [xUnit Test Patterns: Refactoring Test Code](https://www.amazon.co.jp/dp/0131495054/ref=cm_sw_r_tw_dp_U_x_Y8kJCb6EX02F6)
- [オブジェクト指向設計実践ガイド ～Rubyでわかる 進化しつづける柔軟なアプリケーションの育て方](https://gihyo.jp/book/2016/978-4-7741-8361-9)
- [レガシーコード改善ガイド](https://www.shoeisha.co.jp/book/detail/9784798116839)
- [Working Effectively with Legacy Code](https://www.amazon.com/Working-Effectively-Legacy-Michael-Feathers/dp/0131177052)
- [テスト駆動開発](https://www.amazon.co.jp/dp/4274217884/ref=cm_sw_r_tw_dp_U_x_c-CJCb9GKFXC4)
- [実践テスト駆動開発 (Object Oriented SELECTION) ](https://www.amazon.co.jp/dp/4798124583/ref=cm_sw_r_tw_dp_U_x_c.CJCb2G9QMZF)

## 参考記事
- [xUnit Patterns](http://xunitpatterns.com/)
  - [Principles of Test Automation](http://xunitpatterns.com/Principles%20of%20Test%20Automation.html)
  - [Fragile Test](http://xunitpatterns.com/Fragile%20Test.html)
  - [Round trip test](http://xunitpatterns.com/round%20trip%20test.html)
  - [Obscure Tests](http://xunitpatterns.com/Obscure%20Test.html)
  - [Test Utility Methods](http://xunitpatterns.com/Test%20Utility%20Method.html)
  - [layer-crossing test](http://xunitpatterns.com/layer-crossing%20test.html)
  - [SUT](http://xunitpatterns.com/SUT.html)
  - [Conditional Test Logic](http://xunitpatterns.com/Conditional%20Test%20Logic.html)
  - [Test Double](http://xunitpatterns.com/Test%20Double.html)
  - [Dependency Injection](http://xunitpatterns.com/Dependency%20Injection.html)
  - [Dependency Lookup](http://xunitpatterns.com/Dependency%20Lookup.html)
  - [Test-specified Subclass](http://xunitpatterns.com/Test-Specific%20Subclass.html)
- [テストが辛いを解決するテスト駆動開発のアプローチ at PHPカンファレンス仙台2019](https://speakerdeck.com/hgsgtk/tesutokaxin-iwojie-jue-surutesutoqu-dong-kai-fa-falseahuroti-at-phpkanhuarensuxian-tai-2019)
