---
title: "sacloud OSS 2026年8月 ─ sacloud-sdk-go 初リリース、旧 Go SDK 群はアーカイブ"
emoji: "📚"
type: "tech"
topics: ["さくらのクラウド", "OSS", "terraform", "go"]
publication_name: "sakura_internet"
published: false
---

sacloud の OSS 群、毎月あちこちで活発に開発が進んでいるのですが、各リポジトリの Releases を全部追いかけるのは結構大変です。

ということで、2026 年 8 月のリリースから主要なものをまとめてみました。

今月はなんといっても、統合 Go SDK である `sacloud-sdk-go` が初めてタグ付きでリリースされたのが大きなトピックです。あわせて旧 SDK 群のリポジトリがアーカイブされているので、Go から直接さくらのクラウドの API を叩いている方は影響があります。まずはそこから紹介しますね。

## 要注目: sacloud-sdk-go の初リリースと、旧 Go SDK 群のアーカイブ

これまで `iaas-api-go` や `iam-api-go` のように、サービスごとに別々のリポジトリで公開していた Go 向けの SDK を、`sacloud-sdk-go` という 1 つのモノレポに統合しました。2026 年 8 月にその最初のリリースとなる [v0.0.1](https://github.com/sacloud/sacloud-sdk-go/releases/tag/v0.0.1) が出て、下旬には [v0.1.0](https://github.com/sacloud/sacloud-sdk-go/releases/tag/v0.1.0) が続いています。

統合にともなって、`api-client-go` `iaas-api-go` `iaas-service-go` `iam-api-go` `webaccel-api-go` `saclient-go` `packages-go` など **29 個のリポジトリが 2026 年 7 月末をもってアーカイブ済み**になりました。公式のお知らせも出ていますので、これらを直接 import しているコードをお持ちの方はご確認ください。

https://cloud.sakura.ad.jp/news/2026/08/06/migration-to-sacloud-sdk-go-2/

移行先の import パスは、リポジトリ名がそのままサブパッケージになる形です。たとえば `github.com/sacloud/iaas-api-go` は `github.com/sacloud/sacloud-sdk-go/api/iaas`、`github.com/sacloud/iam-api-go` は `github.com/sacloud/sacloud-sdk-go/api/iam` になります。高レベルなサービスライブラリは `service/iaas` `service/webaccel` に入っていて、認証まわりの共通実装は `common/saclient` です。対応表は [README](https://github.com/sacloud/sacloud-sdk-go/blob/main/README.md) にまとまっています。

モノレポ 1 モジュール構成なので、インストールは以下で済みます。

```bash
go get github.com/sacloud/sacloud-sdk-go
```

なお、`autoscaler` や `packer-plugin-sakuracloud`、後述の `sakumock` といった sacloud 側のツールは、今月のリリースで `sacloud-sdk-go` への移行が完了しています。ツールとして使っているぶんには、特に気にしていただく必要はありません。まだ移行が完了していないツールも9月以後、順次対応予定です。

### sacloud-sdk-go 自体の機能追加

[v0.0.1](https://github.com/sacloud/sacloud-sdk-go/releases/tag/v0.0.1) では、さくらのクラウドのリソースを一意に表す Sakura resource name (SRN) を扱う `srn` パッケージが追加されました。

[v0.1.0](https://github.com/sacloud/sacloud-sdk-go/releases/tag/v0.1.0) には、クライアントまわりの追加が 2 つ入っています。ひとつは `common/saclient` の `WithoutProfile()` オプションで、プロファイルの読み込みを明示的に無効化できます。もともと互換性維持のための内部的な仕組みだったものを、正式なオプションとして公開しました。

もうひとつは `EndpointConfig()` からデフォルトゾーンを取得できるようになったことです。アイコンや DNS、GSLB のような全ゾーン共通のリソースを扱うときに、環境変数やプロファイルまで評価した最終的なゾーンの値が必要になるのですが、これまでは saclient からその値を知る手段がありませんでした。`iaas-api-go` を直接使っていた方の移行がしやすくなっているかなと思います。

あわせて `ostype` に Ubuntu 26.04 が追加されました。

### sacloud-sdk-go のバグ修正

[v0.1.0](https://github.com/sacloud/sacloud-sdk-go/releases/tag/v0.1.0) では、fake ドライバの `ProxyLBOp.SetCertificates` で証明書を設定すると panic する不具合と、`Conf` を持たないデータベースに対して `DatabaseOp.GetParameter` を呼ぶと panic する不具合を修正しました。

また、EventBus の `Trigger` / `Schedule` / `ProcessConfiguration` の一覧取得で、エンドポイントのパスによっては種別の絞り込みが効かず、別種別のリソースが混ざって返ることがある不具合も直しています。

## terraform-provider-sakura

今月は [v3.12.8](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.8) と [v3.12.9](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.9) の 2 本が出ています。どちらも VPN ルータまわりの改善が中心です。

### 主要な機能追加

[v3.12.8](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.8) で、`sakura_vpn_router` の `user` 属性にかかっていた最大 100 件の上限を撤廃しました。現状のサービス仕様と合っていない制限だったので、プロバイダ側のバリデーションを外した形です。

### 主要なバグ修正

[v3.12.8](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.8) では、`sakura_vpn_router` の `public_network_interface.aliases` を指定していないときに変換エラーになり、state が壊れてしまう不具合を修正しました。

[v3.12.9](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.9) では、`public_network_interface` の更新可能な属性に再作成を強制する plan modifier がついていた問題を修正しています。これまで値を変えると VPN ルータごと作り直しになっていた属性が、そのまま更新できるようになります。

https://github.com/sacloud/terraform-provider-sakura/releases

## sakumock

さくらのクラウドの API を手元で再現するモックサーバー `sakumock` は、今月 4 本のリリースがあって、対応サービスがぐっと増えました。

[v0.8.0](https://github.com/sacloud/sakumock/releases/tag/v0.8.0) では **CloudHSM** のモックが追加されました。CloudHSM のパーティション、クライアント証明書、IPsec ピア、ソフトウェアライセンスまで一通り扱えます。メールや Webhook で通知を送れる[シンプル通知](https://manual.sakura.ad.jp/cloud/appliance/simplenotification/index.html)のモックも、履歴・ソース・ステータス・ルーティングの並び替えまで実装が進みました。

[v0.9.0](https://github.com/sacloud/sakumock/releases/tag/v0.9.0) ではさらに **Service Endpoint Gateway (SEG)** と **アドオン API** のモックが加わっています。SEG は 2025 年 10 月に提供が始まったスイッチの拡張機能で、プライベートネットワーク内のサーバーから、インターネットを経由せずにオブジェクトストレージなどのマネージドサービスへ接続できるようにするものです ([マニュアル](https://manual.sakura.ad.jp/cloud/network/switch/seg.html))。アドオン API のほうは、AI・CDN・セキュリティ (DDoS 対策 / WAF / 脆弱性検知)・データ分析系まで、11 種類のアドオンすべてに対応しています。

同じく [v0.9.0](https://github.com/sacloud/sakumock/releases/tag/v0.9.0) で、KMS モックに `--key ID=SECRET` オプションが入りました。指定した ID と鍵素材で鍵を起動時に作っておけるので、モックを再起動しても以前暗号化した値をそのまま復号できます。設定ファイルに暗号化した値を置いて起動時に KMS で復号する、といったアプリをローカルで開発するときに便利かなーと思います。[v0.9.1](https://github.com/sacloud/sakumock/releases/tag/v0.9.1) では、この鍵のローテーションも決定的に行えるようになっています。

もうひとつ、[v0.8.0](https://github.com/sacloud/sakumock/releases/tag/v0.8.0) からはモックが **自分の返すレスポンスを OpenAPI 仕様と突き合わせて検証する**ようになりました。違反があってもレスポンス自体は変えず、`GET /_sakumock/spec-violations` で確認できます。実際にこの仕組みで IAM のサービスポリシー系エンドポイントの不整合が見つかって、あわせて修正されています。オブジェクトストレージのモックでは、コントロールプレーンで発行したアクセスキーが S3 データプレーンでもそのまま使えるようになったので、「キーを発行して S3 を叩く」流れを最後まで手元で試せます。

https://github.com/sacloud/sakumock/releases/tag/v0.9.0

## その他の sacloud OSS

`packer-plugin-sakuracloud` は [v0.12.1](https://github.com/sacloud/packer-plugin-sakuracloud/releases/tag/v0.12.1) が出ました。Packer 1.6.0 で廃止された `iso_checksum_url` / `iso_checksum_type` に関する記述をコードとドキュメントから整理しています。このプラグインは `iso_checksum` の値を CD-ROM 名として使っているのですが、いまの `iso_checksum` は `sha256:` のような接頭辞を含むため名前の長さ制限を超えることがあり、切り詰める処理を入れました。あわせて VNC 接続先アドレスの組み立てが正しくない不具合も修正しています。

使用量取得ツールの [sacloud-cpu-usage v0.3.0](https://github.com/sacloud/sacloud-cpu-usage/releases/tag/v0.3.0)、[sacloud-router-usage v0.3.0](https://github.com/sacloud/sacloud-router-usage/releases/tag/v0.3.0)、共通ライブラリの [sacloud-usage-lib v0.2.0](https://github.com/sacloud/sacloud-usage-lib/releases/tag/v0.2.0)、オートスケーラーの [autoscaler v0.20.0](https://github.com/sacloud/autoscaler/releases/tag/v0.20.0) は、いずれも `sacloud-sdk-go` への移行が主な内容です。使い方は変わりませんが、最新版へ上げておくと安心です。

## おわりに

今月は `sacloud-sdk-go` の初リリースと旧 SDK のアーカイブという、Go から API を叩いている方には大きめの変化がありました。移行で困ったことがあれば、ぜひ issue でお知らせください。

`usacloud` と `terraform-provider-sakuracloud` は、それぞれ 2026 年 7 月・2026 年 6 月のリリースが最新版です。こちらも引き続き開発していますので、次のリリースをお待ちください。

気になるものがあれば、ぜひ各リポジトリの Releases を見てみてください。sacloud OSS は GitHub で開発しているので、フィードバックや issue は歓迎です。
