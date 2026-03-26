---
title: "さくらのAI Engineを利用して技術トレンドをキャッチアップするためのアプリを書いてみた"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["さくらのクラウド", "AI Engine", "cloud"]
publication_name: "sakura_internet"
published: true
---

エンジニアとして技術トレンドの把握は重要です。Hacker News なり、はてなブックマークのトレンドなりを見るとよいと思うのですが、そういった情報は流量が多くて読むのが大変。あと、英語の記事はタイトルだけでも日本語になっていてほしい。どれが自分に興味ある記事なのかを選別したい。

みたいな、そういうモチベーションあると思います。

そういうときに使えるアプリを作ってみました。

![alt text](/images/kijiyomu.png)

https://tokuhirom.github.io/kijiyomu/

## どう作ったのか

さくらインターネットでは AI Engine というプロダクトで、OpenAI 互換の chat completion API を提供しています。

https://www.sakura.ad.jp/aipf/ai-engine/

無償枠もあります。これを利用して、日本語訳とスコアリングを行います。

アプリは使い慣れている go 言語で実装しています。go なので、シングルバイナリでアプリケーションが生成されます。
アプリケーションは YAML の設定ファイルを読んで、HTML を生成します。

YAML は以下のように書きます。

```yaml
profile: |
  - 言語/技術: Rust, Go, TypeScript, Python
  - 分野: システムプログラミング, LLM/AIエージェント, ゲーム開発(ブラウザゲーム/Phaser3), 日本語IME, WebAssembly, クラウドインフラ(Kubernetes, docker, etc.), データベース内部構造, コンパイラ/言語処理系
  - 製品: さくらインターネット, OpenAPI, Wails, ogen
  - 趣味: Minecraft, roguelike, ピクセルアート, 演芸
  - 関心低め: スポーツ

feeds:
  - name: Hacker News
    type: hn
    limit: 50

  - name: はてなブックマーク
    type: rdf
    url: https://b.hatena.ne.jp/hotentry/it.rss

  - name: Zenn
    type: rss
    url: https://zenn.dev/feed

  - name: Qiita
    type: atom
    url: https://qiita.com/popular-items/feed

  - name: Classmethod
    type: rss
    url: https://dev.classmethod.jp/feed/

  - name: Lobsters
    type: rss
    url: https://lobste.rs/rss

  - name: Gizmodo JP
    type: rss
    url: https://www.gizmodo.jp/index.xml

  - name: ROOMIE
    type: rss
    url: https://www.roomie.jp/feed/

  - name: Reddit r/rust
    type: reddit
    subreddit: rust

  - name: Reddit r/golang
    type: reddit
    subreddit: golang

  - name: Reddit r/programming
    type: reddit
    subreddit: programming

  - name: Google さくらインターネット
    type: rss
    url: 'https://news.google.com/rss/topics/CAAqJQgKIh9DQkFTRVFvTEwyY3ZNVEl4WW5jeE1HUVNBbXBoS0FBUAE?ceid=JP:ja&oc=3&hl=ja&gl=JP'

  - name: 日本 - 最新 - Google ニュース
    type: rss
    url: 'https://news.google.com/rss/topics/CAAqIQgKIhtDQkFTRGdvSUwyMHZNRE5mTTJRU0FtcGhLQUFQAQ?hl=ja&gl=JP&ceid=JP:ja'
```

実際に生成される HTML がどのようなものかは、以下の URL で確認できます。デモ用にたまに更新しています。
ニュースソースは僕が気になっているようなもので、僕が気になるようなものをスコア高くなるようにしています。

https://tokuhirom.github.io/kijiyomu/

OpenAI の API は [go-openai](https://github.com/sashabaranov/go-openai) を利用していています。

ソースコードは https://github.com/tokuhirom/kijiyomu にあります。


