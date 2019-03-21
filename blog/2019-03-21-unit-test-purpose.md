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

# テスト自動化のゴール
ここで、本エントリのタイトルにもある「ユニットテストの目的」と大きく関係する「テスト自動化のゴール」についてです。『xUnit Test Patterns: Refactoring Test Code』では、6つの目的をあげています。

1. Tests should help us improve quality.
2. Tests should help us understand the SUT.
3. Tests should reduce (and not introduce) risk.
4. Tests should be easy to run.
5. Tests should be easy to write and maintain.
6. Tests should require minimam maintenance as the system evolves around them.

最初の3つの目的は、 **テストがもたらす価値** について示しています。一方、以降の3つは **テスト自身の特徴** について注目したものです。

## Tests should help us improve quality.

## Tests should help us understand the SUT.

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
