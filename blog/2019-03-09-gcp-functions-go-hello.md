Google Cloud Functions Go

# Cloud Functions
Cloud Functions でGoを書くことができます。
現在ベータ版の機能です。
主に２つの関数がある
HTTP Functions
Background Functions

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

# Goを書いていく

# refs
そのほか、Google Cloud Functions Go について、こちらの記事も参考になります。

- [BetaリリースされたGoのGoogle Cloud Functionsを試す #gcp #golangjp - budougumi0617](https://budougumi0617.github.io/2019/01/17/try-go-on-google-cloud-functions/
)
- [Cloud FunctionsにGoがサポートされたので使ってみた on Qiita](https://qiita.com/Morix1500/items/09a660c77dcf1e7a064b)
