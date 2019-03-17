# TL;DR
- 自動ユニットテストの12個の原則を『[xUnit Test Patterns: Refactoring Test Code](https://www.amazon.co.jp/dp/0131495054/ref=cm_sw_r_tw_dp_U_x_Y8kJCb6EX02F6)』からおさえる
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

テストを維持したり、ドキュメントとしてテストを利用するたびにテストを再考する必要が出てくるため、余分に時間がかかってしまいます。

[Test Utility Methods](http://xunitpatterns.com/Test%20Utility%20Method.html)のライブラリを使用すると、詳細をすべてコーディングする必要がないため、テストの作成コストが下がります。また、Test Utility Methodにまとめることでテストケースの意図が伝わりやすいものになります。

## Don't Modify the SUT（テスト対象システムを修正しない）
効果的なテストでは、アプリケーションの一部を[Test Double](http://xunitpatterns.com/Test%20Double.html)に置き換えるか、[Test-specified Subclass](http://xunitpatterns.com/Test-Specific%20Subclass.html)を使用して動作の一部をオーバーライドすることが求められます。テスト対象システムが依存するコンポーネントから返される値など（[indirect input](http://xunitpatterns.com/indirect%20input.html)）を制御したり、SUTから別コンポーネントのメソッド呼び出しなど（[indirect output](http://xunitpatterns.com/indirect%20output.html)）を傍受して動作検証を行う必要があるためです。また、テスト環境では許容できない副作用や依存関係があるといった理由になります。

テスト対象システムを変更することは、[Test Hooks](http://xunitpatterns.com/Test%20Hook.html)や[Test-specified Subclass](http://xunitpatterns.com/Test-Specific%20Subclass.html)での振る舞いのオーバーライド、依存オブジェクトを[Test Double](http://xunitpatterns.com/Test%20Double.html)に置き換えていたとしても、危険なことです。

私たちは、ソフトウェアが本番環境で使用されるような構成をもって、ソフトウェアをテストしていることを確認する必要があります。
テスト対象システムを取り巻くコンテキストをより制御するために、Test Double など何かに置き換える必要がある場合、本番の動きに代表されるものであることを確認しておく必要があります。

## Keep Tests Independent（テストを独立させる）
テストが相互依存していて、さらに順序に依存している場合は、テストの失敗から得られる有用なフィードバックが得にくくなります。相互依存しているようなテストが複数同時に失敗した場合、何が問題なのかわかりにくい状況に陥ってしまいます。

独立したテスト（*Independent Test*）は、単独で実行できます。テスト対象を検証できる状態にするためには、テストケースごとに用意されたフィクスチャ（[Fresh Fixture](http://xunitpatterns.com/Fresh%20Fixture.html)）を設定します。

[Fresh Fixture](http://xunitpatterns.com/Fresh%20Fixture.html)は、複数テストで共有されるフィクスチャ[Shared Fixture](http://xunitpatterns.com/Shared%20Fixture.html)と比較するとはるかに相互に独立しているテストになる可能性が高いです。

独立したテストを書くことで、ユニットテストの失敗から原因を突き止めやすい状況を生み出すことができます。

## Isolate the SUT（SUTを隔離する）
もし、テスト対象のSUTが他のソフトウェアに依存している場合、他のソフトウェアの動作の変更によってテストが突然失敗することがあります。たとえば、SUTが外部システムに依存しているようなケースです。
これは、 *Context Sensitivity* と呼ばれる壊れやすいテストのひとつです。

これを避けるために、テストを完全に制御しながら依存関係の可能性のあるすべての反応をソフトウェアに注入できる必要があります。具体的には、依存関係のあるソフトウェアを[Dependency Injection](http://xunitpatterns.com/Dependency%20Injection.html)や[Dependency Lookup](http://xunitpatterns.com/Dependency%20Lookup.html)、[Test-specified Subclass](http://xunitpatterns.com/Test-Specific%20Subclass.html)を用いた一部の動作のオーバーライドによって、[Test Double](http://xunitpatterns.com/Test%20Double.html)に置き換える方法によって制御できます。

## Minimize Test Overlap（テストの重複を最小限に抑える）
**機能に対して可能な限りテストを少なくするようにテストを構成する** 必要があります。頻繁にテストすることでテストカバレッジを改善したいかもしれませんが、同じ機能を検証するテストは通常同時に失敗します。

そのため、SUTの機能変更に対して、複数ヵ所に同じメンテナンスをする必要が出てきます。このように複数のテストで同じ機能を検証することは、テストのメンテナンスコストを上げる可能性があり、品質をあまり向上させない可能性があります。

## Minimize Untestable Code（テスト不可能なコードを最小限に抑える）
完全自動化テストでのテストの難しいケースがあります。たとえば、GUIコンポーネントやマルチスレッドのコードや、テストメソッド自体があります。
テスト不可能なコードは自動化テストでの保護が難しく、安全なリファクタリング・機能追加が困難なものです。

保守が必要なテスト不可能なコードの量を最小限に抑えることが望ましいです。そのためには、テストしたいロジックを、テスト不可能なクラスの外に移動するリファクタリングを行うことです。

*Minimize Untestable Code* によって、テストカバレッジが改善され、結果コードに対する自信とリファクタリングする能力が向上します。

## Keep Test Logic Out of Production Code（テストロジックをプロダクションコードから除外する）
テスト容易性が確保されていないプロダクションコードの場合、テストをしやすくするための *hook* をプロダクションコードに入れたい誘惑があります。

```php
<?php
declare(strict_types=1);

final class TestHook
{
    public $mode;

    public function exec()
    {
        if ($this->mode === 'test') {
            // exec to logic for testing
        } else {
            // exec to logic for production
        }
    }
}
```

しかし、テストはシステムの動作を検証することです。テスト中のシステムの動作が異なる場合、テストで本番動作を確認する目的は達成できません。
プロダクションコードには、 `if testing then` といった条件付きステートメントを含めるべきではありません。

## Verify One Condition per Test（テストごとにひとつの条件を検証する）
自動テストでは、テストスクリプトを記述することになりますが、それらは単一のテスト条件を検証する必要があります。なぜなら、自動テストでは、アサーションが一回失敗するだけでテストの実行が停止し、残りのテストでは何がうまくいって何がうまくいかないのかというデータがテストから得られないためです。

### 1つのテストケースに1つのアサーション？
ただし、 *Verify One Condition per Test* の「One Condition」をどう意味づけるかによって意見が異なることがあります。それは、「ひとつのテストケースに対してひとつのアサーションのみ」という主張です。

たしかに、「テストごとに1つのアサーション」とすることで、テストメソッドの命名が簡単になる利点があります。しかし一方、多くのアサーションが必要な場合、多くのテストケースが必要になります。複数のアサーションメソッドの呼び出しを1つに呼び出しに減らすような[Custom Assertion](http://xunitpatterns.com/Custom%20Assertion.html)を利用することでこのアプローチは、テストを読みやすくなる可能性がありますが、そうではない場合、「1つのテストケースに1つのアサーション」という主張は強制されるものではありません。

## Test Concerns Separately（懸念は別々にテストする）
複雑なアプリケーションの動作は多数の小さい動作の集合から構成されますが、動作の一部は同じコンポーネントによって提供されることがあります。これに対して、単一のテストメソッドで複数の懸念事項をテストすることは、いずれかの懸念事項の変更時に破綻してしまうことが問題点としてあります。さらにつらい状況は懸念事項が問題になっているか明らかにならないことです。

その場合、テストケースの失敗に対するトラブルシューティングと修正が難しくなります。

複数ある懸念をそれぞれ1つのテストケースとしてテストすることで、特定の部分に問題があることをテストの失敗が明確に教えてくれます。

## Ensure Commensurate Effort and Responsibility（調和の取れた努力と責任の確保）
テストを書いたり修正するのにかかる努力の量は、対応する機能を実装するのに要する努力を超えるべきではありません。

### 補足：テスト初級者
テストの作成・修正に関する努力の量に関して、特にテストを書き始めたテスト初級者にとっては、とくに多く感じることになります。この原則に照らし合わせるのであれば、即座に「テストを書かない」という方向に対して議論したくなるかもしれません。

しかし、その場合これまでの自動テストに関する原則を満たしていないテストを書いている可能性があります。その時点で感じる努力量を減らすための解決策は、これまでの原則を踏まえた上で、「テストをうまくなる」ことです。

書籍『[オブジェクト指向設計実践ガイド ～Rubyでわかる 進化しつづける柔軟なアプリケーションの育て方](https://gihyo.jp/book/2016/978-4-7741-8361-9)』では、次の言葉で説明されています。

「*テストにコストがかかることの解決方法は、テストをやめることではありません。うまくなることです。*」

# まとめ
[xUnit Test Patterns: Refactoring Test Code](https://www.amazon.co.jp/dp/0131495054/ref=cm_sw_r_tw_dp_U_x_Y8kJCb6EX02F6)から自動ユニットテストにおける12個の原則を見ていきました。普段ユニットテストを作成している中で、方法論に迷った場合は一度原則に立ち返ってみてはいかがでしょうか。

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
  - [Test Hooks](http://xunitpatterns.com/Test%20Hook.html)
  - [Data-Driven Test](http://xunitpatterns.com/Data-Driven%20Test.html)
  - [indirect input](http://xunitpatterns.com/indirect%20input.html)
  - [indirect output](http://xunitpatterns.com/indirect%20output.html)
  - [Fresh Fixture](http://xunitpatterns.com/Fresh%20Fixture.html)
  - [Shared Fixture](http://xunitpatterns.com/Shared%20Fixture.html)
  - [Custom Assertion](http://xunitpatterns.com/Custom%20Assertion.html)
- [テストが辛いを解決するテスト駆動開発のアプローチ at PHPカンファレンス仙台2019](https://speakerdeck.com/hgsgtk/tesutokaxin-iwojie-jue-surutesutoqu-dong-kai-fa-falseahuroti-at-phpkanhuarensuxian-tai-2019)
