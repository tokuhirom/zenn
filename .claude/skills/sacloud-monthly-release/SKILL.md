---
name: sacloud-monthly-release
description: github.com/sacloud/ org の当月リリースノートを集約して Zenn 記事の下書き (articles/<slug>.md, published:false) を生成する。terraform-provider-sakura(cloud) と usacloud を中心に、その他 OSS の主要な機能追加と主要なバグ修正を紹介する文体で書く。月次のリリース振り返り記事を書きたいときに使う。
---

# sacloud 月次リリースまとめ記事生成

`github.com/sacloud/` の OSS 群について、指定月のリリースを集約して Zenn 記事の下書きを書く。著者はさくらインターネットの sacloud OSS 担当者として、自分の publication (`sakura_internet`) で紹介する立場。

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
  -q '.[] | select(.archived==false) | .name'
```

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

## 記事の書き方

### frontmatter (固定値)

```yaml
---
title: "sacloud OSS の YYYY 年 M 月リリースまとめ"
emoji: "📚"
type: "tech"
topics: ["さくらのクラウド", "OSS", "terraform", "cli"]
publication_name: "sakura_internet"
published: false
---
```

`published: false` は固定。ユーザがレビューしてから自分で `true` に変える運用。

### 本文の構成

1. **リード文** (3-5 行) — 月次まとめである旨を軽い口調で。「sacloud の OSS 群、毎月地味に動いているのですが、どこで何が動いたのか追うのが大変なので、まとめてみました。」のようなトーン
2. **terraform-provider-sakura / terraform-provider-sakuracloud** セクション
   - `### 主要な機能追加` / `### 主要なバグ修正` のサブ見出し
   - 各項目は箇条書き or 短い段落。技術的に何が嬉しいかを 1-2 行添える
   - リリースへの bare URL を末尾に貼る (Zenn がカード化する)
3. **usacloud** セクション
   - 同じ構造
4. **その他の sacloud OSS** セクション
   - `*-api-go` 群、`sakuracloud_exporter`、`packer-plugin-sakuracloud`、`iaas-service-go`、`saclient-go` など
   - 1 リポジトリにつき 1-3 行で要約
   - リリースが多い repo は箇条書き、少ない repo はまとめて段落で
5. **締め** (2-3 行) — 「気になるものがあればぜひ各リポジトリの Releases を見てみてください」「フィードバックや issue は歓迎です」的な OSS 担当者の口調で

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
