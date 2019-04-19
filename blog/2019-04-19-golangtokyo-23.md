# golang.tokyo #23 参加レポート
## TL;DR
- golang.tokyo #23に参加

平成最後のgolang.tokyoに参加してきました。LT枠で落選してしまったのですが、LT枠は落選しても参加できるとのことでお邪魔しました。

https://golangtokyo.connpass.com/event/126673/

## 絶対に分かるポインタ / tenntenn

https://docs.google.com/presentation/d/1u93oMe7QForbqrCwGE2lHbu99t4I9_m5IzfeYETD7BU/edit#slide=id.p

### Memo
- 変数
  - メモリ上で管理されている値を保持する領域
- ポインタ
  - 変数の格納先を表す値、どこにしまってあるか
- 型の表現方法
- 型リテラルのポイント型
  - ex. `*struct{N int}`
- ポインタのポインタ
  - `**int`: `*int`型のポインタ
- ポインタ演算
  - `&`: ポインタ値取得
  - `*`: ポインタが指す先取得
- ポインタが必要な理由
  - 代入
- 内部にポインタ
  - slice, map, channel

## Goをはじめるにあたって知っておいてほしいツールやテスト/mom0tomo, micchiebear
### Memo
- golang.orgを見る、golang.jpは内容が古い
  - https://golang.org/
  - 人にシェアするときも気をつけようね...
- golangweeklyが有用な情報もらえる
  - https://golangweekly.com/
  - https://twitter.com/golangweekly
- `go env`

```
-> % go env GOROOT
/usr/local/opt/go/libexec
kazukihigashiguchi@BASE-P
```

## Delveを用いたデバッグ & pprofを用いたプロファイリング / 渡邉 光
### Memo
- debug
  - https://github.com/go-delve/delve
  - https://rat.cis.k.hosei.ac.jp/article/devel/debugongccgdb1.html
- profiling
  - 実行時間、ヒープ使用量の把握
  - https://golang.org/pkg/net/http/pprof/
## 感想
tenntennさんのポインタの話がわかっているつもりでもいざ質問されるとしっかり間違えたので、まだまだ基礎を抑え直さないといけないなと反省しました。あと、ベースターズビールがおいしかったです！
