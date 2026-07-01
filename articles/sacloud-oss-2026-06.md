---
title: "sacloud OSS 2026年6月 ─ Go SDK 群を sacloud-sdk-go に統合、v2 プロバイダー終了へ"
emoji: "📚"
type: "tech"
topics: ["さくらのクラウド", "OSS", "terraform", "cli"]
publication_name: "sakura_internet"
published: true
---

sacloud の OSS 群、毎月活発に開発されておりまして、各リポジトリの Releases を全部追いかけるのは結構大変です。

ということで、2026 年 6 月のリリースから主要なものをまとめてみました。今月は Go SDK 群を `sacloud-sdk-go` に統合していくお知らせや、`terraform-provider-sakuracloud` (v2) のメンテナンス終了アナウンス、API モックスイート `sakumock` の対応サービス拡充など、大きめのトピックが続いた月でした。`terraform-provider-sakura` と `usacloud` を中心に、その他の OSS にも触れていきます。

## Go SDK 群を sacloud-sdk-go に統合します

まずは大事なお知らせから。さくらのクラウドの Go 向け SDK は、これまで `iaas-api-go` や `kms-api-go` のように API ごとに別々のリポジトリで提供してきたのですが、これらを単一の [sacloud-sdk-go](https://github.com/sacloud/sacloud-sdk-go) に統合していくことになりました ([お知らせ](https://cloud.sakura.ad.jp/news/2026/06/18/migration-to-sacloud-sdk-go/))。

個別 SDK が増えてきてメンテナンスの手間が大きくなってきたので、ひとつにまとめて管理しやすくしよう、という狙いです。

スケジュールとしては、**2026 年 7 月末に旧 SDK のメンテナンスを終了し、リポジトリをアーカイブする**予定です。それ以降、新機能の追加やバグ修正は `sacloud-sdk-go` の側だけで行われます。対象は `addon-api-go`・`api-client-go`・`apprun-api-go`・`iaas-api-go`・`iaas-service-go`・`iam-api-go`・`kms-api-go`・`monitoring-suite-api-go`・`object-storage-api-go`・`saclient-go`・`secretmanager-api-go`・`simplemq-api-go`・`webaccel-api-go`・`workflows-api-go` など、全 27 リポジトリにのぼります。

これらの Go SDK をアプリケーションから直接 import して使っている方は、`sacloud-sdk-go` への移行をご検討ください。なお `terraform-provider-sakura` や `usacloud` といったツールは、内部の SDK が順次差し替わっていくだけなので、利用者側で特別な対応は必要ありません。

https://cloud.sakura.ad.jp/news/2026/06/18/migration-to-sacloud-sdk-go/

## terraform-provider-sakura / terraform-provider-sakuracloud

今月は `terraform-provider-sakura` (v3) が [v3.12.0](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.0) から [v3.12.4](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.4) まで、`terraform-provider-sakuracloud` (v2) が [v2.36.0](https://github.com/sacloud/terraform-provider-sakuracloud/releases/tag/v2.36.0) と [v2.36.1](https://github.com/sacloud/terraform-provider-sakuracloud/releases/tag/v2.36.1) のリリースでした。

### v2 プロバイダーのメンテナンス終了について

今月の一番のお知らせです。[v2.36.1](https://github.com/sacloud/terraform-provider-sakuracloud/releases/tag/v2.36.1) のドキュメント更新で、`terraform-provider-sakuracloud` (v2) のメンテナンス終了がアナウンスされ、v3 プロバイダー (`terraform-provider-sakura`) への移行が推奨される形になりました。`terraform-provider-sakura` (v3) 側でも [v3.12.4](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.4) で v2 のメンテナンス終了と移行ノートの更新が行われています。

これまで数か月かけて v3 へのリソース移植を進めてきて、移植対象のリソースがひととおり揃ったので、いよいよ v2 から v3 へ、という段階に入った形です。v2 をお使いの方は、移行ガイドを見ながら少しずつ v3 への切り替えを検討していただければと思います。

### 主要な機能追加

- **オブジェクトストレージのアカウント自動作成** に対応しました ([v3.12.0](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.0))。Terraform だけでオブジェクトストレージを管理していると、まだアカウントが存在しないサイトに対して操作してしまうことがあります。そういう場合に、自動でアカウントを作成してから処理を進めるようになりました。

- **サービスプリンシパルキーの KID を provider 設定でも指定可能** になりました ([v3.12.0](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.0))。これまで環境変数では KID (キー ID) を指定できていたのですが、`provider` ブロックなど他の設定方法では指定できませんでした。今回これが揃った形です。

### 主要なバグ修正

今月の v3 は、各リソースの `terraform import` 周りの不具合をまとめて直したのが中心でした。

- シークレットマネージャ・シンプル監視・ウェブアクセラレータのバケットオリジン・VPN ルーター・NoSQL・モニタリングスイートのアラートで、`terraform import` がうまくいかない不具合を修正しました ([v3.12.2](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.2), [v3.12.4](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.4))。インポート時に nil panic が起きたり、複合 ID のインポートに対応していなかったりといったケースに該当します。
- KMS リソースの不具合を修正しました ([v3.12.1](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.1))。
- エンハンスドデータベース (OndemandDB) で、ホスト名の扱いによって毎回差分が出てしまう問題を修正しました ([v3.12.3](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.3))。plan 時に state 側のホスト名を使うようにしています。
- AppRun 専有型で nil pointer panic が起きる不具合を修正しました ([v3.12.4](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.4))。

なお v3・v2 の両方で、コンテナレジストリの `access_level` (公開アクセス設定) が deprecated になりました ([v3.12.2](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.2), [v2.36.0](https://github.com/sacloud/terraform-provider-sakuracloud/releases/tag/v2.36.0))。コンテナレジストリの非公開化対応に伴うものです。`access_level` を使っている方は、今後の挙動の変更にご注意ください。

https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.4

## usacloud

今月の `usacloud` は [v1.22.2](https://github.com/sacloud/usacloud/releases/tag/v1.22.2) から [v1.22.4](https://github.com/sacloud/usacloud/releases/tag/v1.22.4) まで、バグ修正中心のリリースでした。

### 主要なバグ修正

- `config delete` で現在選択中 (`current`) のプロファイルを削除したときに、`current` が `default` に戻らない不具合を修正しました ([v1.22.4](https://github.com/sacloud/usacloud/releases/tag/v1.22.4))。内部の SDK を `saclient-go` に移行した際に抜けてしまっていた挙動を、元どおりに戻した形です。
- プロファイルの読み込みやクライアント初期化まわりで、設定によってはエラーになってしまうケースを修正しました ([v1.22.4](https://github.com/sacloud/usacloud/releases/tag/v1.22.4))。

terraform-provider 側と同じく、コンテナレジストリの `access-level` も deprecated になっています ([v1.22.2](https://github.com/sacloud/usacloud/releases/tag/v1.22.2))。

## その他の sacloud OSS

### sakumock ─ さくらのクラウド API のモックスイート

今月いちばん動きが大きかったのが `sakumock` です。`sakumock` は、さくらのクラウド API のローカルモックサーバー群です。実際のクラウドにつながずに、ローカルで各サービスの API を立ち上げて開発やテストに使えます (まだ開発中の Alpha なので、仕様は予告なく変更される可能性があります)。

使い方が 2 通りあるのがいいところで、用途に応じて選べます。

- **Go ライブラリとして組み込む**: 各サービスがインプロセスで起動できるテストサーバーとして提供されているので、Go のテストコードからサクッと立ち上げて、SDK の動作確認に使えます。
- **CLI / Docker イメージとして使う**: 単一の `sakumock` バイナリで `sakumock all` を実行すると、全サービスが 1 プロセスでまとめて立ち上がります。`sakumock env` を使うと、SDK や Terraform プロバイダーが参照する接続先の環境変数を dotenv 形式で吐き出してくれるので、エンドポイントを手書きする必要がありません。コンテナイメージも ghcr.io で配布されています。

これまでは SimpleMQ・シークレットマネージャ・KMS・シンプル通知あたりに対応していたのですが、6 月の一連のリリースで対応サービスが一気に増えました。

- [v0.3.0](https://github.com/sacloud/sakumock/releases/tag/v0.3.0): モニタリングスイートのモックを追加。あわせて ghcr.io でマルチプラットフォームのコンテナイメージを配布するようになりました。
- [v0.4.0](https://github.com/sacloud/sakumock/releases/tag/v0.4.0): EventBus とオブジェクトストレージのモックを追加。オブジェクトストレージは versitygw を使った S3 データプレーンもオプションで使えます。環境変数をまとめて出力する `sakumock env --export` も入りました。
- [v0.5.0](https://github.com/sacloud/sakumock/releases/tag/v0.5.0): IAM・AppRun・AppRun 専有型のモックを追加。
- [v0.6.0](https://github.com/sacloud/sakumock/releases/tag/v0.6.0): Workflows のモック (コントロールプレーン + Runbook 実行エンジン) を追加。EventBus から SimpleMQ・シンプル通知への連携 (サービスリンク) も動くようになりました。

Terraform との結合動作も確認されているので、さくらのクラウドを使った開発・テストで「いちいち本物の API を叩きたくない」という場面で便利だと思います。

https://github.com/sacloud/sakumock

### sacloud-otel-collector

[v0.7.0](https://github.com/sacloud/sacloud-otel-collector/releases/tag/v0.7.0) がリリースされました。ベースの OpenTelemetry Collector を v0.154.0 にアップグレードしています。アップグレード手順は [UPGRADE_v0.6_to_v0.7.md](https://github.com/sacloud/sacloud-otel-collector/blob/main/docs/UPGRADE_v0.6_to_v0.7.md) にまとまっているので、更新する方は目を通しておくとよさそうです。

あわせて sacloud exporter が `http://` のエンドポイントを受け付けるようになり、安定度が alpha に引き上げられました。

### monitoring-suite-api-go

[v0.2.1](https://github.com/sacloud/monitoring-suite-api-go/releases/tag/v0.2.1) で、通知ルーティング (notification routing) で match ラベルが空の場合をサポートするようになりました。

### iaas-api-go

[v1.29.2](https://github.com/sacloud/iaas-api-go/releases/tag/v1.29.2) で、コンテナレジストリの公開アクセス設定が deprecated になりました。terraform-provider や usacloud 側の `access_level` deprecated と対になる変更です。

## おわりに

2026 年 6 月の sacloud OSS は、Go SDK 群の `sacloud-sdk-go` への統合アナウンス、`terraform-provider-sakuracloud` (v2) のメンテナンス終了アナウンス、`sakumock` の対応サービス拡充と、移行・統合まわりのトピックが多い月でした。Go SDK を直接お使いの方は `sacloud-sdk-go` への移行を、v2 プロバイダーをお使いの方は v3 への移行を、それぞれ少しずつ検討していただければと思います。

気になるものがあれば、ぜひ各リポジトリの Releases を見てみてください。sacloud OSS は GitHub で開発しているので、フィードバックや issue は歓迎です。
