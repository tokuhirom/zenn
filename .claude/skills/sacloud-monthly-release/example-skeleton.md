---
title: "sacloud OSS の YYYY 年 M 月リリースまとめ"
emoji: "📚"
type: "tech"
topics: ["さくらのクラウド", "OSS", "terraform", "cli"]
publication_name: "sakura_internet"
published: false
---

<!--
このファイルは骨格テンプレです。実際の記事生成時にはこの構造を参考にして、
収集した release notes から具体的な内容を埋めて記事を作成してください。
最終的な記事ファイル (articles/<slug>.md) には、この HTML コメントは含めないでください。
-->

<!-- リード文: 月次まとめである旨をやさしい口調で 3-5 行 -->

sacloud の OSS 群、毎月地味に動いているのですが、各リポジトリの Releases を全部追いかけるのは結構大変です。

ということで、YYYY 年 M 月のリリースから主要なものをまとめてみました。`terraform-provider-sakura` と `terraform-provider-sakuracloud`、`usacloud` を中心に、その他の OSS にも触れていきます。

## terraform-provider-sakura / terraform-provider-sakuracloud

<!-- 両プロバイダの当月リリースを統合して紹介。
     v3 (sakura) と既存 (sakuracloud) を区別しつつ、機能追加・バグ修正に分けて書く。 -->

### 主要な機能追加

<!-- 例:
- 〜リソースに対応しました。これまで Web UI からしか操作できなかったのですが、これで terraform で管理できるようになります。
- 〜のフィールドが追加されました。〜のような場面で便利です。

各項目の最後にリリース URL を bare で貼る:
https://github.com/sacloud/terraform-provider-sakura/releases/tag/vX.Y.Z
-->

### 主要なバグ修正

<!-- 例:
- 〜のときにエラーになる問題を修正しました。
-->

## usacloud

<!-- usacloud の当月リリースを紹介。 -->

### 主要な機能追加

### 主要なバグ修正

## その他の sacloud OSS

<!-- 以下を 1 リポジトリにつき 1-3 行で紹介。リリースが多い repo は箇条書きでも可。
     対象例:
     - iaas-api-go / iaas-service-go
     - 各種 *-api-go (workflows, addon, iam, cloudhsm, security-control, apigw, secretmanager, kms, simplemq, eventbus, object-storage, dedicated-storage, service-endpoint-gateway, monitoring-suite, apprun, apprun-dedicated, simple-notification など)
     - sakuracloud_exporter (Prometheus exporter)
     - packer-plugin-sakuracloud
     - saclient-go / sacloud-sdk-go
     - packages-go
     リリースがなかった repo は触れない。
-->

### API ライブラリ群

<!-- 例:
- `monitoring-suite-api-go` v0.x.y がリリースされました。〜に対応しています。
  https://github.com/sacloud/monitoring-suite-api-go/releases/tag/v0.x.y
-->

### その他

<!-- exporter / packer plugin / sdk など -->

## おわりに

今月もたくさんのリリースがありました。気になるものがあれば、ぜひ各リポジトリの Releases を見てみてください。

sacloud OSS は GitHub で開発しているので、フィードバックや issue は歓迎です。
