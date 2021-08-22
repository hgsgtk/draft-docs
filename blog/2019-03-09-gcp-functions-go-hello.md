# TL;DR
- 2019年1月17日にGoogle Cloud FunctionsがGo1.11をサポート
- Google Cloud Functions Goの環境構築方法
- HTTP・background functions の２つの使い方を抑える

# Google Cloud Functions とは
![画像：cloudfunctiongo](blog/gcp-intro.png)

Google Cloud Functions とは、イベント駆動型アプリケーションを作成する際に使用できる、Google Cloud Platformが提供しているサービスです。Cloud Functionsの形式でコードを記述することでロジックを組み立てられます。
これまで、Node.js・Pythonがサポートされていましたが、2019年1月17日にGo1.11をベータ版としてサポートしました。

https://cloud.google.com/blog/products/application-development/cloud-functions-go-1-11-is-now-a-supported-language

本記事では、Google Cloud Functions Go の基本的な使い方を見ていきます。

# 目次
- 環境構築
- イベント: HTTP Functions
- イベント: Background Functions
- refs

# 環境構築
今回の環境は次の環境です。
go 1.11
gcloud

## go
今回は、冒頭にも述べていますがGo言語を利用するため、goをインストールしておきましょう。

https://cloud.google.com/go/docs/setup
## gcloud
gcloudが必要になる。
インストールする
https://cloud.google.com/sdk/docs/

ベータ機能を使えるようにするためにアップデートしてベータ版を使えるようにする


```shell
$ gcloud components update
$ gcloud components install beta
```

次のような出力結果が出たら完了です。

```shell
╔════════════════════════════════════════════════════════════╗
╠═ Creating update staging area                             ═╣
╠════════════════════════════════════════════════════════════╣
╠═ Installing: gcloud Beta Commands                         ═╣
╠════════════════════════════════════════════════════════════╣
╠═ Creating backup and activating new installation          ═╣
╚════════════════════════════════════════════════════════════╝

Performing post processing steps...done.

Update done!
```

### Go Modulesの有効が必要
また、Go Modulesの利用を有効化するために、次の環境変数を設定します。

```shell
$ export GO111MODULE=on
```

# ２つの使い方
現在ベータ版の機能です。
主に２つの関数がある
HTTP Functions
Background Functions

# イベント: HTTP Functions
HTTP functionsは、HTTP要求によって呼び出される関数です。標準の `net/http` ライブラリの `http.HandlerFunc` 型にしたがって記述します。たとえば、次のような関数があるとします。

```go:function.go
package function

import (
	"fmt"
	"net/http"
)

func HelloWorld(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json")
	fmt.Fprint(w, "Hello World")
}
```

これを、Google Cloud Functionsにデプロイしてみます。
これをコードとしてデプロイするのみでHTTPを受け付けて処理するエンドポイントがリリースされました。 API Gateway なしで、HTTPSのURLが提供されます。

```shell
// curlする
```

# イベント: Background Functions
イベント

## ログを出力する
ログの出し方。

# イベント： Background Functions
Background Functionsを試す。

# refs
そのほか、Google Cloud Functions Go について、こちらの記事も参考になります。

- [BetaリリースされたGoのGoogle Cloud Functionsを試す #gcp #golangjp - budougumi0617](https://budougumi0617.github.io/2019/01/17/try-go-on-google-cloud-functions/
)
- [Cloud FunctionsにGoがサポートされたので使ってみた on Qiita](https://qiita.com/Morix1500/items/09a660c77dcf1e7a064b)
