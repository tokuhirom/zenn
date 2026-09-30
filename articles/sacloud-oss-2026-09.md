---
title: "sacloud OSS 2026年9月 ─ Networking Suite / AppRun、OTel v0.8"
emoji: "📚"
type: "tech"
topics: ["さくらのクラウド", "OSS", "terraform", "cli"]
publication_name: "sakura_internet"
published: true
---

sacloud の OSS 群、毎月あちこちで活発に開発が進んでいるのですが、各リポジトリの Releases を全部追いかけるのは結構大変です。

ということで、2026 年 9 月のリリースから主要なものをまとめてみました。

今月は `terraform-provider-sakura` と `sacloud-sdk-go` に Networking Suite 対応が入り、AppRun のシークレットも Terraform から扱えるようになりました。OpenTelemetry Collector にはアップグレード時に確認しておきたい変更も入っています。

## terraform-provider-sakura

今月は [v3.13.0](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.13.0) から [v3.14.2](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.14.2) まで、4 本のリリースがありました。

### 主要な機能追加

[v3.13.0](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.13.0) で、Networking Suite の `subnet_group`、`subnet`、`interface_connection` リソースと、サブネット・サブネットグループのデータソースが追加されました。Networking Suite はサブネットグループの中にサブネットを作成し、サーバーのインターフェースを接続して使うネットワーク機能です ([マニュアル](https://manual.sakura.ad.jp/cloud/network/networking-suite/about.html))。これまでコントロールパネルや API で行っていた構成を Terraform で管理しやすくなります。

[v3.13.0](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.13.0) では、`sakura_apprun_shared` のシークレットにも対応しました。環境変数とは別に、アプリケーションへ渡す値をシークレットとして管理できます。

[v3.13.0](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.13.0) では `sakura_enhanced_lb_acme` の更新もサポートしました。既存リソースを import したあと、証明書の設定を更新しやすくなっています。

[v3.14.0](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.14.0) では、Service Endpoint Gateway (SEG) で SimpleAI を利用する構成に対応しました。SEG はプライベートネットワーク内のサーバーから、対応するマネージドサービスへ接続するためのスイッチの拡張機能です ([マニュアル](https://manual.sakura.ad.jp/cloud/network/switch/seg.html))。

[v3.14.1](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.14.1) では、`sakura_apprun_dedicated_version` に write-only の環境変数属性が追加されました。`value_wo` と `value_wo_version` を使うことで、環境変数の値を Terraform state に残さずに更新できます。

### 主要なバグ修正

[v3.14.1](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.14.1) では、VPN ルータの standard プランでサポートされない `public_network_interface` を apply 後ではなく plan 時に検出するようになりました。

[v3.14.2](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.14.2) では、VPN ルータの `user` に Terraform の unknown 値を指定したときに変換エラーになる問題と、import 時に `user` が state へ復元されない問題を修正しました。ワークフローの `CreatedAt` に関するリグレッションも修正されています。

https://github.com/sacloud/terraform-provider-sakura/releases

## sacloud-sdk-go

### 主要な機能追加

[v0.2.0](https://github.com/sacloud/sacloud-sdk-go/releases/tag/v0.2.0) で Networking Suite API が追加されました。サブネットグループ、サブネット、インターフェース接続、アドレスを Go から操作できます。また、クライアントのレートリミットを `WithAPIRequestRateLimit` オプションでプログラムから指定できるようになりました。

同じ [v0.2.0](https://github.com/sacloud/sacloud-sdk-go/releases/tag/v0.2.0) では AppRun 共用型の OpenAPI v1.5.0 に追従し、コンポーネントシークレットの作成・更新・取得に対応しました。シークレットの値は API レスポンスに含めず、キーだけを扱う仕様です。

[v0.3.0](https://github.com/sacloud/sacloud-sdk-go/releases/tag/v0.3.0) では SimpleMQ の API 定義が更新され、キューの一覧・取得・更新、メッセージの送受信、API キーの再発行などのクライアントが更新されています。CloudHSM、KMS、シークレットマネージャーの API 定義も更新され、新しい API やパラメータを扱えるようになりました。

また、[v0.3.0](https://github.com/sacloud/sacloud-sdk-go/releases/tag/v0.3.0) では SEG が SimpleAI を扱えるようになり、IP アドレスのホスト名について SDK 側で FQDN 形式を強制しないようになりました。

### 主要なバグ修正

[v0.3.0](https://github.com/sacloud/sacloud-sdk-go/releases/tag/v0.3.0) では、IP アドレスのホスト名を FQDN として検証していた問題を修正しました。PTR レコードに登録する文字列など、FQDN 以外の形式も API 側の検証に任せられます。

https://github.com/sacloud/sacloud-sdk-go/releases

## sakumock

さくらのクラウド API を手元で再現するモックサーバー `sakumock` では、今月 [v0.10.0](https://github.com/sacloud/sakumock/releases/tag/v0.10.0) と [v0.11.0](https://github.com/sacloud/sakumock/releases/tag/v0.11.0) がリリースされました。

[v0.10.0](https://github.com/sacloud/sakumock/releases/tag/v0.10.0) からは、各サービスの README や Terraform のサンプルをバイナリに埋め込み、`sakumock docs` で参照できるようになりました。`docs <topic>`、`--search`、`--section` なども使えるので、ソースコードを手元に置かずにモックの使い方を確認できます。

[v0.11.0](https://github.com/sacloud/sakumock/releases/tag/v0.11.0) では AppRun 共用型のコンポーネントシークレットに対応しました。SDK とモックサーバーを同じ仕様で試せるようになっています。

https://github.com/sacloud/sakumock/releases

## sacloud-otel-collector v0.8.0 の変更点

[v0.8.0](https://github.com/sacloud/sacloud-otel-collector/releases/tag/v0.8.0) は破壊的変更を含むリリースです。アップグレード前に [UPGRADE_v0.7_to_v0.8.md](https://github.com/sacloud/sacloud-otel-collector/blob/v0.8.0/docs/UPGRADE_v0.7_to_v0.8.md) を確認してください。

主な追加は、ログとトレースの送信キューを `sending_queue.storage` で永続化できるようになったことです。Collector を再起動してもキューに残っているデータを失いにくくなるため、ストレージ拡張と組み合わせた運用ができます。メトリクスの送信キューは永続化の対象外です。

あわせて selfmetrics receiver が生成していた synthetic scrape metrics が削除され、OpenTelemetry Collector 自体も v0.161.0 に更新されています。

https://github.com/sacloud/sacloud-otel-collector/releases/tag/v0.8.0

## usacloud

2026 年 9 月は `usacloud` のリリースはありませんでした。

## おわりに

今月は Networking Suite と AppRun のシークレット対応が Terraform と Go SDK の両方に入り、構成管理やローカル開発で試しやすくなりました。`sacloud-otel-collector` を利用している方は、アップグレード手順を確認してから更新してみてください。

気になるものがあれば、ぜひ各リポジトリの Releases を見てみてください。sacloud OSS は GitHub で開発しているので、フィードバックや issue は歓迎です。
