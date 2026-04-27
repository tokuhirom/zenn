# 文体ガイド (sacloud 月次リリースまとめ)

著者の既存記事 (`articles/ai-engine-kijiyomu.md`, `articles/ansible-collection-inventory-sacloud.md`, `books/7c2c7820b2c36c/`) から抽出。

## 基本方針

- **ですます調で統一** する。常体 (〜だ／〜である) を混ぜない
- **やさしい口調** を心がける。読み手に話しかけるような距離感
- 一文を長くしすぎず、短く区切って読みやすくする
- 専門用語は使うが、難しい言い回しで威圧しない
- 「自分はこう思う」「こうしてみました」のような **個人の体験ベース** の語り口を混ぜる

## 真似してよい言い回し

冒頭・導入:

- 「〜があると思います。」「〜みたいな、そういうモチベーションあると思います。」
- 「〜なのですが、〜」と接続して背景を説明する
- 「そういうときに使えるアプリを作ってみました。」のような結び

紹介・説明:

- 「〜というプロダクトで、〜を提供しています」(さくらインターネットの事業を紹介する文脈)
- 「〜を利用して、〜を行います。」
- 「〜してみました」「〜書いてみました」(自分の取り組みを謙虚に紹介)
- 「便利かなーと思います」「いいかなと思います」(やわらかい主観)

締め・誘導:

- 「ご利用ください。」
- 「ぜひ〜してみてください。」
- 「気になる方は〜を見てみてください。」
- 「フィードバックや issue は歓迎です。」

## 句読点・記号の使い方

- 句点は全角「。」
- 読点は基本「、」だが、半角「､」を意図的に混ぜている箇所もある (元記事を踏襲)
- 英数字の前後に半角スペースを入れる (例: `terraform-provider-sakura の v3 が`)
- バッククォートでコマンド・変数名・リポジトリ名を囲む (例: `terraform-provider-sakura`)

## 見出しの付け方

- セクション見出しは内容そのものを示す具体的な言葉にする
  - OK: `## terraform-provider-sakura / terraform-provider-sakuracloud`
  - NG: `## 概要` `## 詳細` `## まとめ` (機械的で著者の記事には出てこない)
- サブ見出しは `### 主要な機能追加` `### 主要なバグ修正` のように何が書いてあるか一目でわかる形に

## 外部リンク

**インラインを優先**。文中の自然な単語にリンクを張る形が基本:

```
[v3.8.0](https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.8.0) で〜が追加されました。
[Service Endpoint Gateway 対応](https://github.com/sacloud/.../releases/tag/v3.9.0) も入っています。
```

**bare URL (Zenn のカード化)** は、1 セクションあたり 1 個までに抑える:

- 「このセクションで一番伝えたいリリース」もしくはリポジトリトップを 1 本だけ
- bare URL を 3 本も 4 本も連続で並べるのは見た目がうるさいので避ける

例 (悪い):
```
https://github.com/sacloud/foo/releases/tag/v1
https://github.com/sacloud/foo/releases/tag/v2
https://github.com/sacloud/foo/releases/tag/v3
```

例 (良い):
```
今月は [v1](URL)、[v2](URL)、[v3](URL) と 3 本のリリースがありました。
代表として最新の v3 を置いておきます。

https://github.com/sacloud/foo/releases/tag/v3
```

## frontmatter

固定で以下を使う:

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

`published: false` は必ず守る (下書きで出してユーザにレビューしてもらう)。

## 真似しないこと (NG パターン)

- ですます調と常体の混在 (例: 「〜です。〜だ。」)
- 「概要」「結論」「ポイント」のような無機質な見出し
- 「本記事では〜について述べる」のような硬い書き出し
- 全リリースを表組みで羅列するだけの構成
- 各リリースの changelog をコピペで貼って終わり (必ず要約・意味付けをする)
- 過剰な絵文字・記号装飾

## 例: リード文のサンプル

> sacloud の OSS 群、毎月地味に動いているのですが、各リポジトリの Releases を全部追いかけるのは結構大変です。
>
> ということで、今月のリリースから主要なものをまとめてみました。terraform-provider-sakura(cloud) と usacloud を中心に、その他の OSS にも触れていきます。

## 例: セクション内の説明サンプル

> v3.1.0 がリリースされました。今回は X リソースに対応しています。これまでは Web UI からしか操作できなかったのですが、これで terraform で管理できるようになります。
>
> https://github.com/sacloud/terraform-provider-sakura/releases/tag/v3.1.0

## 例: 締めのサンプル

> 今月もたくさんのリリースがありました。気になるものがあれば、ぜひ各リポジトリの Releases を見てみてください。
>
> sacloud OSS は GitHub で開発しているので、フィードバックや issue は歓迎です。
