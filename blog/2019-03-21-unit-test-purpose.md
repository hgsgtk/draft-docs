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
テストは、テストの読み手に対して **「コードがどのように機能するか」** を示せます。

### Tests as Documentation（"ドキュメント"としてのテスト）


## Tests should reduce (and not introduce) risk.

## Tests should be easy to run.

## Tests should be easy to write and maintain.

## Tests should require minimam maintenance as the system evolves around them.

# Next
『[xUnit Test Patterns: Refactoring Test Code](https://www.amazon.co.jp/dp/0131495054/ref=cm_sw_r_tw_dp_U_x_Y8kJCb6EX02F6)』では、 **自動ユニットテストの原則** ・ **自動ユニットテストにおけるアンチパターン** についても言及されています。
こちらについても次のURLで手前味噌ながらまとめましたので、合わせてご参照ください。

- [xUnit Test Patternsから学ぶ12個のユニットテストの原則 on Qiita](https://qiita.com/hgsgtk/items/a3186a250d36d3b224d9)
- [xUnit Test Patternsから学ぶテストアンチパターン on Speaker Deck](https://speakerdeck.com/hgsgtk/testing-anti-pattern-learned-in-xunit-test-pattern)

# References
## Books
- [xUnit Test Patterns: Refactoring Test Code](https://www.amazon.co.jp/dp/0131495054/ref=cm_sw_r_tw_dp_U_x_Y8kJCb6EX02F6)

## Articles
- [Xunit Test Patterns](http://xunitpatterns.com/)
  - [Fragile Test](http://xunitpatterns.com/Fragile%20Test.html)
- [社内LT20170810(xUnit Test Patterns Chapter3について)](https://speakerdeck.com/o0h/she-nei-lt20170810) by [@o0h_](https://twitter.com/o0h_)
- [xUnit Test Patternsから学ぶ12個のユニットテストの原則 on Qiita](https://qiita.com/hgsgtk/items/a3186a250d36d3b224d9) by [@hgsgtk](https://twitter.com/hgsgtk)
- [xUnit Test Patternsから学ぶテストアンチパターン on Speaker Deck](https://speakerdeck.com/hgsgtk/testing-anti-pattern-learned-in-xunit-test-pattern) by [@hgsgtk](https://twitter.com/hgsgtk)
- [PHPバージョンアップと決済リプレイスを支えたユニットテスト #phpcon](https://speakerdeck.com/hgsgtk/phpbaziyonatuputojue-ji-ripureisuwozhi-etayunitutotesuto-number-phpcon) by [@hgsgtk](https://twitter.com/hgsgtk)
