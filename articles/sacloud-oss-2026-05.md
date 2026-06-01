---
title: "sacloud OSS 2026年5月 ─ WebAccel・CDROM が Terraform 対応、ELB に新機能追加"
emoji: "📚"
type: "tech"
topics: ["さくらのクラウド", "OSS", "terraform", "cli"]
publication_name: "sakura_internet"
published: true
---

sacloud の OSS 群、毎月地味に動いているのですが、各リポジトリの Releases を全部追いかけるのは結構大変です。

ということで、2026 年 5 月のリリースから主要なものをまとめてみました。今月は `terraform-provider-sakura` が中心で、v3 プロバイダーへのリソース移植が続いています。

## terraform-provider-sakura

今月は [v3.10.0](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.10.0) と [v3.11.0](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.11.0) の 2 本がリリースされました。

### 主要な機能追加

- **ウェブアクセラレータ (WebAccel) に対応** しました ([v3.10.0](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.10.0))。ウェブアクセラレータはさくらのクラウドの CDN サービスで、コンテンツ配信の高速化や負荷分散に使えるプロダクトです。サイト (`sakura_webaccel`)・ACL (`sakura_webaccel_acl`)・証明書 (`sakura_webaccel_certificate`)・有効化設定 (`sakura_webaccel_activation`) など一通りのリソースが追加されています。v2 プロバイダーには既にあった機能で、v3 への移植が完了した形です。

- **CDROM (ISO イメージ) リソースに対応** しました ([v3.10.0](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.10.0))。`sakura_cdrom` リソースと `sakura_cdrom` データソースが追加され、サーバーに ISO イメージをマウントする構成を Terraform で管理できるようになっています。こちらも v2 からの移植です。

- **`sakura_enhanced_lb` に `origin_guard` と `strict_rule` 属性が追加** されました ([v3.11.0](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.11.0))。`origin_guard` はオリジンサーバーへの不正アクセスを防ぐためのトークン認証設定 (半角英数 8〜32 文字)、`strict_rule` はルールの厳格化設定です。エンハンスドロードバランサーのセキュリティ設定をより細かく Terraform で管理できるようになりました。

### 主要なバグ修正

- `sakura_enhanced_lb` の `health_check` パラメータに不要なバリデーションが設定されていた問題を修正しました ([v3.10.0](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.10.0))。v2 では問題なく動いていたパラメータが v3 で弾かれるケースがあったので、正しい挙動に戻されています。

- `sakura_enhanced_lb_acme` リソースで `origin_guard` と `strict_rule` が設定されていなかった問題を修正しました ([v3.11.0](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.11.0))。ACME (Let's Encrypt) 対応の Enhanced LB でも同様の設定が使えます。

https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.11.0

## その他の sacloud OSS

### IaaS API ライブラリ群

`iaas-api-go` の [v1.29.1](https://github.com/sacloud/iaas-api-go/releases/tag/v1.29.1) では、エンハンスドロードバランサーのプラン変更時に `BackendHttpKeepAlive`・`MonitoringSuiteLog`・`OriginGuard`・`StrictRule` の 4 フィールドが失われていた不具合を修正しました。`ChangeProxyLBPlan` を使ったプラン変更後に設定が意図せずリセットされていたケースに該当します。

### saclient-go

[v0.4.0](https://github.com/sacloud/saclient-go/releases/tag/v0.4.0) で、プロファイル操作 (`NewProfileOp`) の初期化タイミングが遅延されるようになりました。これまでは `$HOME` ディレクトリが存在しない環境 (一部のコンテナ環境や CI など) でも構造体の生成時に `$HOME` を参照してしまっていました。今回の変更により、環境変数や他の方法で API キーを設定している場合は `$HOME` がなくても正常に動作するようになっています。

### apprun-dedicated-api-go

[v0.2.1](https://github.com/sacloud/apprun-dedicated-api-go/releases/tag/v0.2.1) で、ラッパーパッケージのエクスポート済み構造体フィールドすべてに JSON タグが追加されました。このライブラリの構造体を JSON にシリアライズしたい場面 (ログ出力、外部システムへのデータ連携など) で使いやすくなっています。

## おわりに

2026 年 5 月の sacloud OSS は、`terraform-provider-sakura` でのリソース移植が一段落した月でした。ウェブアクセラレータと CDROM の追加で、移植する予定のあるリソースはひととおり v3 に揃った形です。v2 にしかないリソースが多少残っていますが、そちらは当面移植の予定はない見込みです。

気になるものがあれば、ぜひ各リポジトリの Releases を見てみてください。sacloud OSS は GitHub で開発しているので、フィードバックや issue は歓迎です。
