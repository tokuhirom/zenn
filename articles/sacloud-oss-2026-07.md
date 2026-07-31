---
title: "sacloud OSS 2026年7月 ─ auth-status API 廃止予告、usacloud や exporter は要更新"
emoji: "📚"
type: "tech"
topics: ["さくらのクラウド", "OSS", "terraform", "cli"]
publication_name: "sakura_internet"
published: false
---

sacloud の OSS 群、毎月あちこちで活発に開発が進んでいるのですが、各リポジトリの Releases を全部追いかけるのは結構大変です。

ということで、2026 年 7 月のリリースから主要なものをまとめてみました。今月は `terraform-provider-sakura` と `usacloud` は落ち着いていて、バグ修正が中心でした。そのぶん `sacloud-otel-collector` や `sakumock` あたりが活発に動いています。

あと、OSS 利用者にとって影響の大きい **API 廃止のお知らせ** が 1 件出ています。ここが一番大事なので、最初に触れておきますね。

## 要注意: `auth-status` API の廃止予告

さくらのクラウドから、`GET /api/cloud/1.1/auth-status` API を **2026 年 9 月 30 日（水）** に廃止するというお知らせが出ています（当初は 9 月 1 日でしたが、9 月 30 日に延期されました）。

これがなぜ OSS 利用者に関係するかというと、以下のプロダクトの **古いバージョン** がこの API に依存しているためです。

- `usacloud`
- `sakuracloud_exporter`
- `iaas-service-go`

いずれも新しいバージョンでは依存を外していますので、上記を使っている方は **廃止日までに最新版へアップデート** しておいてください。古いバージョンのまま 2026 年 9 月 30 日を迎えると、認証まわりで動かなくなる可能性があります。

詳細は公式のお知らせをご確認ください。

https://cloud.sakura.ad.jp/news/2026/07/06/api-get-api-cloud-1-1-auth-status-discontinuation/

## terraform-provider-sakura / terraform-provider-sakuracloud

今月は `terraform-provider-sakura` (v3) 側で 3 本のリリースがありました（[v3.12.5](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.5)、[v3.12.6](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.6)、[v3.12.7](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.7)）。`terraform-provider-sakuracloud` (v2) 側は 7 月のリリースはありませんでした。

内容としては地味なバグ修正が中心ですが、import まわりで挙動が変わった箇所があるので、そこだけ先に触れておきます。

### 主要な変更

- import 時のリソース指定の区切り文字を `/` に統一しました（[v3.12.7](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.7)）。これまで `_` を使っていたリソースでも `/` が使えるようになっています。従来の書き方も引き続きサポートされているので、いますぐ書き換える必要はありませんが、これから import する方は `/` 区切りに寄せておくとよいと思います。

### 主要なバグ修正

- `object_storage` のパーミッションが import できない問題を修正しました（[v3.12.6](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.6)）。
- `vpn_router` の firewall ルールの順序が内部的に不定になり、`terraform apply` のときに state と config の inconsistency エラーが出ることがある問題を修正しました（[v3.12.6](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.6)）。内部的に `Set` を使う実装に変えていますが、設定の書き方はこれまでと互換なので、利用者側での対応は不要です。
- `vpn_router` の `firewall.logging` に付いていたデフォルト値が原因で、意図しない plan の差分が出る問題を修正しました（[v3.12.7](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.7)）。
- Service Endpoint Gateway (SEG) を import する際に、ゾーンの指定が必要になりました（[v3.12.6](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.6)）。仕組み上、別のゾーンから読み込むことができないための変更です。
- `workflows` リソースの update で、最新リビジョンの runbook が更新されていないときに状態が壊れることがある問題を修正しました（[v3.12.7](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.7)）。
- `monitoring_suite` の `alert_rule` リソースの不具合とサンプルを修正しました（[v3.12.5](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.12.5)）。

## usacloud

`usacloud` は今月 [v1.22.5](https://github.com/sacloud/usacloud/releases/tag/v1.22.5) から [v1.22.8](https://github.com/sacloud/usacloud/releases/tag/v1.22.8) まで出ていますが、大半は CI まわりの調整でした。利用者から見て嬉しい変更は 1 つです。

### 主要な機能追加

- 新しいバージョンが出ているかをチェックするリリースアラートを、環境変数 `USACLOUD_NO_VERSION_CHECK` で抑制できるようになりました（[v1.22.5](https://github.com/sacloud/usacloud/releases/tag/v1.22.5)）。CI やスクリプトの中で `usacloud` を回していて、アラート表示を一時的に止めたい、といった場面で便利かなと思います。

## その他の sacloud OSS

### sacloud-otel-collector

`sacloud-otel-collector` は、さくらのクラウドのモニタリングスイートにメトリクス・ログ・トレースを送るための OpenTelemetry Collector のディストリビューションです。今月は [v0.7.1](https://github.com/sacloud/sacloud-otel-collector/releases/tag/v0.7.1) から [v0.7.6](https://github.com/sacloud/sacloud-otel-collector/releases/tag/v0.7.6) まで、かなり活発に動きました。

主だったところを挙げると、

- Kafka exporter を追加しました（[v0.7.1](https://github.com/sacloud/sacloud-otel-collector/releases/tag/v0.7.1)）。収集したテレメトリを Kafka へ流せるようになっています。
- リリースビルドに Windows / macOS のバイナリを追加しました（[v0.7.1](https://github.com/sacloud/sacloud-otel-collector/releases/tag/v0.7.1)）。これまで Linux 中心だったのですが、手元の Windows / macOS でも動かしやすくなりました。
- Windows のイベントログを取り込む `windowseventlogreceiver` を追加しました（[v0.7.2](https://github.com/sacloud/sacloud-otel-collector/releases/tag/v0.7.2)）。
- コレクター自身の状態を監視するための selfmetrics receiver を追加しました（[v0.7.4](https://github.com/sacloud/sacloud-otel-collector/releases/tag/v0.7.4)）。

Windows 対応がまとめて入った月なので、Windows 上で監視エージェントを動かしたい方は一度試してみるとよさそうです。

https://github.com/sacloud/sacloud-otel-collector/releases/tag/v0.7.6

### sakumock

`sakumock` は、さくらのクラウド API のモックサーバーです。実際のクラウドを叩かずにツールやスクリプトのテストをしたいときに使えるもので、今月 [v0.7.0](https://github.com/sacloud/sakumock/releases/tag/v0.7.0) と [v0.7.1](https://github.com/sacloud/sakumock/releases/tag/v0.7.1) が出ています。

- OpenAPI 仕様からリクエストボディのバリデーションを生成し、各サービスに展開しました（[v0.7.0](https://github.com/sacloud/sakumock/releases/tag/v0.7.0)）。実サーバーに近い形で、おかしなリクエストを弾いてくれるようになっています。
- API Gateway のモック（コントロールプレーン / データプレーン）を追加しました（[v0.7.0](https://github.com/sacloud/sakumock/releases/tag/v0.7.0)）。OIDC 認証やリクエスト/レスポンスの変換、CORS などにも対応しています。
- OpenTelemetry のトレースに対応しました（[v0.7.0](https://github.com/sacloud/sakumock/releases/tag/v0.7.0)）。
- 意図的にエラーを注入する fault injection を追加しました（[v0.7.1](https://github.com/sacloud/sakumock/releases/tag/v0.7.1)）。異常系のテストがしやすくなります。

### API ライブラリ群

- `service-endpoint-gateway-api-go` [v0.2.0](https://github.com/sacloud/service-endpoint-gateway-api-go/releases/tag/v0.2.0) で、DNS プライベートホストゾーンが無効なときにサーバーから短い応答が返るケースに対応しました。これまでこのレスポンスをうまく扱えなかったのを修正したものです。
- `monitoring-suite-api-go` [v0.2.2](https://github.com/sacloud/monitoring-suite-api-go/releases/tag/v0.2.2) で、`log_measure_rule` の OpenAPI discriminator まわりの不具合を修正しました。

## おわりに

今月は機能追加というより、廃止予告への備えと足まわりの修正が中心の月でした。特に `auth-status` API の廃止は、古いバージョンの `usacloud` / `sakuracloud_exporter` / `iaas-service-go` を使っている方に影響しますので、2026 年 9 月 30 日までに最新版へのアップデートをおすすめします。

気になるものがあれば、ぜひ各リポジトリの Releases を見てみてください。sacloud OSS は GitHub で開発しているので、フィードバックや issue は歓迎です。
