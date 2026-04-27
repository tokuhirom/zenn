---
name: sacloud-monthly-release
description: github.com/sacloud/ org の当月リリースノートを集約して Zenn 記事の下書き (articles/<slug>.md, published:false) を生成する。terraform-provider-sakura(cloud) と usacloud を中心に、その他 OSS の主要な機能追加と主要なバグ修正を紹介する文体で書く。月次のリリース振り返り記事を書きたいときに使う。
---

# sacloud 月次リリースまとめ記事生成

`github.com/sacloud/` の OSS 群について、指定月のリリースを集約して Zenn 記事の下書きを書く。著者はさくらインターネットの **sacloud OSS 担当者**として、自分の publication (`sakura_internet`) で紹介する立場。「OSS 担当として今月の動きを紹介する」という立ち位置で書くこと。

## いつ使うか

ユーザが「sacloud の月次リリースまとめ書いて」「今月の sacloud リリース記事作って」「先月の sacloud リリースをまとめたい」のように依頼してきたとき。

## 対象月の決め方

- 引数なし → 現在の年月 (`date +%Y-%m`)
- ユーザが `2026-03` のような YYYY-MM を渡したらその月
- 「先月」「今月」のような相対指定もそのまま解釈

決定した月をユーザに一文で報告してから収集に入る (例: 「2026 年 4 月のリリースを集めます」)。

## データ収集

すべて `gh` コマンド経由。書き込み系は使わない。

### 1. アクティブなリポジトリ列挙

```bash
gh api orgs/sacloud/repos --paginate \
  -q '.[] | select(.archived==false) | select(.private==false) | .name'
```

**private repo は必ず除外する。** `gh` で認証されているとアクセス可能な private repo も返ってくるが、記事は公開記事なので private は触れない。`.private==false` の絞り込みは省略しないこと。

### 2. 各 repo のリリース一覧を並列取得

リポジトリ数が多い (40+) ので、Bash の並列実行で取得する:

```bash
gh release list -R sacloud/<repo> \
  --json tagName,publishedAt,name --limit 30
```

`publishedAt` を対象月でフィルタ。対象月にリリースがない repo はその時点で除外。

### 3. 採用候補のリリース本文を取得

```bash
gh release view <tag> -R sacloud/<repo> \
  --json body,tagName,publishedAt,url
```

## フィルタ規則 (スキップ判定)

以下のいずれかに該当するリリースは記事に載せない:

- 本文の changelog 行が **dependabot/renovate のバンプのみ** (`Bump X from a to b` の繰り返し、`@dependabot` がほぼすべて)
- 本文が `chore:` `ci:` `docs:` `test:` `style:` の prefix だけで構成されている
- changelog 本文が空 / `Full Changelog` リンクのみ
- パッチリリースで `typo` `README` `comment` 修正のみ

採用したリリースから抽出する行:

- `feat:` / `feature:` / `add:` → **主要な機能追加** に分類
- `BREAKING CHANGE` / `breaking:` → 機能追加の中で目立たせる (「破壊的変更」と明示)
- `fix:` / `bugfix:` → **主要なバグ修正** に分類
- 上記 prefix がない場合も、文面から機能追加 or バグ修正を判断して分類

dependabot 由来の `chore(deps):` は基本スキップだが、**主要依存 (terraform plugin sdk, hashicorp/terraform-plugin-framework, go の major version 等) のメジャーバンプ**は触れてよい。

### 内部実装の変更には触れない

以下のような **内部実装の差し替え** は記事に含めない (利用者の視点で何も変わらないため):

- `saclient-go` への移行、`api-client-go` 依存の削除といった内部 SDK の差し替え
- ogen (OpenAPI generator) への移行など、コード生成手法の変更
- FTP クライアントライブラリの差し替え (例: `sacloud/ftps` → `jlaffaye/ftp`)
- 内部リファクタリング・モジュール分離・lint 対応・vulncheck 対応
- `NewClient` の引数の型を別 SDK に合わせて統一する (non-Breaking) 系の調整

利用者から見て **新しいことができるようになった / 既存の挙動が変わった** 変更だけを採用する。

### API バージョンアップの「中身」を深掘りする

リリースノートに「`X API v1.3.0` に対応」「OpenAPI spec v1.3.0 を使用」のような記述があったら、それだけで満足せず **PR 本文や OpenAPI 差分から、追加された具体的なエンドポイント・メソッド・フィールドを特定する**:

```bash
# リリースに含まれる PR を確認
gh pr view <PR番号> -R sacloud/<repo> --json title,body,files

# OpenAPI 仕様書が含まれている場合は diff も確認
gh pr diff <PR番号> -R sacloud/<repo> -- 'openapi/**' 'apis/**/oas_*.go'
```

たとえば「IAM API v1.3.0 に対応」とだけ書かれていたら、PR 本文から「IAM ロールに `lowest_grantable_resource` が追加」「ユーザープロビジョニング (`/scim-configurations`) API が追加」のような具体的な追加内容を抽出して記事に書く。

## 記事の書き方

### frontmatter

```yaml
---
title: "sacloud OSS YYYY年M月まとめ ─ <今月のハイライト>"
emoji: "📚"
type: "tech"
topics: ["さくらのクラウド", "OSS", "terraform", "cli"]
publication_name: "sakura_internet"
published: false
---
```

`published: false` は固定。ユーザがレビューしてから自分で `true` に変える運用。

### タイトルの付け方

はてなブックマーク・X (Twitter) など SNS で目立たせたいので、**今月のハイライトをタイトルに含める**。「リリースまとめ」だけだと素通りされるので、具体的に何が起きた月なのかを示す。

書き方の指針:

- 前半に「sacloud OSS YYYY年M月」、`─` (全角ダッシュ) で区切ったあとに今月の目玉を 1〜3 個並べる
- ハイライトは **読者が「お、これは知っておきたい」と思うもの** を選ぶ:
  - 新プロダクトへの Terraform / SDK 対応
  - 主要 API のメジャーアップデート (SCIM 対応、認証系の刷新など、実務インパクトの大きいもの)
  - 破壊的変更 (アップデート時に注意すべきもの)
- **全長は 70 文字以内**。Zenn の制約 (タイトル最大 70 文字) があるので超えると保存に失敗する。書き上げたら必ず文字数をカウント (`echo -n "$title" | awk '{print length}'`) して確認する
- 動詞は短く: 「対応」「追加」「リリース」「公開」など
- プロダクト名は **公式略称があれば使う** (例: Service Endpoint Gateway → SEG、Service Endpoint Gateway API ライブラリは略さない)

例:
- `sacloud OSS 2026年4月 ─ SEG / AppRun 専有型が Terraform 対応、IAM に SCIM 追加` (66 文字)
- `sacloud OSS 2026年5月 ─ usacloud v2 リリース、モニタリングスイートに新メトリクス`

避けるパターン:
- `sacloud OSS 2026年4月のリリースまとめ` (素っ気ない)
- `【保存版】sacloud 完全ガイド` (煽り系・著者の作風と合わない)
- 70 文字を超えるもの (Zenn が拒否する)

### 本文の構成

1. **リード文** (3-5 行) — 月次まとめである旨を軽い口調で。「sacloud の OSS 群、毎月地味に動いているのですが、どこで何が動いたのか追うのが大変なので、まとめてみました。」のようなトーン
2. **terraform-provider-sakura / terraform-provider-sakuracloud** セクション
   - `### 主要な機能追加` / `### 主要なバグ修正` のサブ見出し
   - 各項目は箇条書き or 短い段落。技術的に何が嬉しいかを 1-2 行添える
   - **個別バージョンへのリンクは文中にインラインで** `[v3.8.0](https://github.com/sacloud/.../releases/tag/v3.8.0)` のように貼る
   - セクション末尾に **そのリポジトリ全体の代表 URL** (リリースページや repo トップ) を bare で 1 つだけ置いて Zenn のカード化に使う
3. **usacloud** セクション
   - 同じ構造
4. **その他の sacloud OSS** セクション
   - `*-api-go` 群、`sakuracloud_exporter`、`packer-plugin-sakuracloud`、`iaas-service-go`、`saclient-go` など
   - 1 リポジトリにつき 1-3 行で要約
   - リリースが多い repo は箇条書き、少ない repo はまとめて段落で
   - リンクはインライン優先。bare URL を貼るのは「特に注目してほしい新規リポジトリ」など限定的な場面のみ
5. **締め** (2-3 行) — 「気になるものがあればぜひ各リポジトリの Releases を見てみてください」「フィードバックや issue は歓迎です」的な OSS 担当者の口調で

### リンクの貼り方の方針

- **インライン優先** — `[v3.8.0](URL)` `[Service Endpoint Gateway 対応](URL)` のように、文中の自然な単語にリンクを張る
- **`vX.Y.Z` などのバージョン番号 (GitHub リリースタグに対応するもの) は、本文中に出てくるたびに毎回インラインリンクにする**。`(v3.8.1)` のような括弧書きや、リード文・他セクションからの後方参照も例外なくリンク化する
  - 例外: `IAM API v1.3.0` `OpenAPI spec v1.3.0` のような **上流の API/仕様のバージョン**は GitHub リリースに対応しないのでリンクしない
- **bare URL (Zenn カード化) は 1 セクションあたり 1 個まで** — 視線が散らかるので連発しない。代表的なリリース 1 本、もしくはリポジトリトップを置く
- Bare URL を連続で並べるレイアウト (旧スタイル) は **使わない**

### 具体例を出すときはコードを確認してから

「たとえば〜を検出してくれます」のような **具体的な動作例を記事に書くときは、必ず該当する OSS の実コード (rego ファイル、テストファイル、ソースコード) を `gh api repos/.../contents/...` などで読んでから書く**。リリースノートや README の概説だけで具体例を捏造しない。

公式ドキュメントが存在する場合 (例: `docs.usacloud.jp/terraform-policy/...`) は、そちらの言い回しに寄せたほうが正確で読み手にも親切。

### PR タイトルだけで feat / fix を判断しない

リリースノートに `feat:` `Support X` `Add Y` のような **機能追加風のタイトルが並んでいても、PR 本文を読むと実態はバグ修正だったケースがある**。

例: saclient-go v0.3.6 の "Support AcceptLanguage option in CompatSettingsFromAPIClientOptions" は、タイトルだけ読むと機能追加に見えるが、PR 本文には「`AcceptLanguage` がサポートされていないために一部のケースでエラーになっていた問題を修正」と書かれており、実態はバグ修正。

採用したリリースの **PR 本文 (`gh pr view <番号> -R <repo> --json body`) を全部読み、何が動機・経緯だったかを把握してから書く**。タイトルの体裁 (feat: / fix: / chore:) は補助情報として扱う。

利用者から見た時、その変更が:

- **これまでできなかったことができるようになった** → 機能追加
- **これまでも (使い方によっては) できたが壊れていた / 期待通り動かなかったのを直した** → バグ修正

として書き分ける。

### バグ修正はさらっと書く

バグ修正は **事実を 1 行で書く** のが基本。「〜という不具合を修正しました」で十分。

修正の副産物 (内部的に追加された API、ついでに直した周辺機能、新しい設定方法など) を **「使えるようになりました」と機能としてプッシュ調で並べない**。著者が利用者に積極的に使ってほしいと思っていない補助的な機能まで強調すると、「使ってほしい機能」と読者に誤解される。修正の経緯を解説したくなっても、ぐっとこらえて短く書く。

例 (悪い):

> v0.3.6 では `SAKURA_ACCEPT_LANGUAGE` 環境変数を指定するとクラッシュする不具合を修正しました。原因は `CompatSettingsFromAPIClientOptions` で `AcceptLanguage` がサポートされていなかったことで、これに対応したことで環境変数経由でも設定できるようになっています。

例 (良い):

> v0.3.6 では `SAKURA_ACCEPT_LANGUAGE` 環境変数を指定したときにクラッシュする不具合を修正しました。

明確な機能追加 (新しい API メソッド、新リソース、新コマンドなど) は逆に **何が嬉しいかを 1〜2 文で書く** ほうがよい。バグ修正と機能追加で書き方の温度感を変える。

### 新規・マイナーなプロダクトは軽く紹介する

新しく登場したプロダクトや、まだ広く知られていないプロダクト (例: Service Endpoint Gateway, AppRun 専有型, シンプル通知, シークレットマネージャ など) を初めて記事内で取り上げるときは、**1〜2 文で「何のサービスか」を補足する**。読者が「OSS リリースを見にきたら知らないプロダクト名が出てきた」状態にならないようにする。

調べ方:

- まず `manual.sakura.ad.jp` や `cloud.sakura.ad.jp/news/` `cloud.sakura.ad.jp/products/` を WebSearch / WebFetch で確認
- 公式の説明 (1 文) を引いて、自分の言葉でやさしく言い直す
- 補足の最後に出典の URL を `([プロダクトページ](URL))` のような形でインラインで添える

書き方の例:

> Service Endpoint Gateway に対応しました。SEG は 2025 年 10 月に提供開始されたスイッチの拡張機能で、プライベートネットワーク内のサーバーからオブジェクトストレージや AI Engine などのマネージドサービスへ、インターネットを経由せずにプライベート接続できるようにするものです ([プロダクトページ](URL))。

すでに何度も記事で取り上げているプロダクト (terraform-provider-sakura 自体、usacloud、IAM など、文脈で自明なもの) には補足を入れない。

### 文体の参考

詳細は `style-guide.md` を参照。**生成前に必ず一度読むこと**。要点:

- **ですます調で統一**。常体 (〜だ／〜である) を絶対に混ぜない
- **やさしい口調**。読み手に話しかけるような距離感で書く
- 短い段落 (1-3 文)、一文を長くしすぎない
- 「〜してみました」「便利かなーと思います」「ご利用ください」「ぜひ〜してみてください」のような著者語彙を時々混ぜる
- 「本記事では〜について述べる」のような硬い書き出しは避ける
- 表組みで全リリース羅列する形式は **避ける** — 文章で紹介する
- 生成後、全文を見直して「ですます調が崩れていないか」「やさしい口調になっているか」を確認する

骨格テンプレは `example-skeleton.md` を参照。

## 出力手順

1. `cd /Users/to-matsuno/work/zenn && npx zenn new:article` でファイルを scaffold (zenn-cli が hash の slug を生成)
2. 生成された `articles/<slug>.md` の内容を上記 frontmatter + 本文で **完全に上書き**
3. 上書き後、自分で全文を読み直し、以下を確認する:
   - ですます調で統一されているか (常体が混ざっていないか)
   - やさしい口調になっているか (硬い・上から目線になっていないか)
   - フィルタ規則に違反するリリース (dependabot 単独など) が混入していないか
4. 完了後、生成パスをユーザに報告し「frontmatter の `published: false` のまま下書きとして残しています。内容を確認のうえ `true` に変えてください」と添える

## 避けること

- AI 生成感のある硬い見出し (「概要」「結論」「まとめ」など機械的なもの) — 著者の既存記事には出てこない
- ですます調と常体の混在
- リリースを表組みでただ羅列する構成
- 各 release の changelog をそのまま貼り付ける (要約と意味付けをすること)
- `published: true` で出力する (常に下書き)
