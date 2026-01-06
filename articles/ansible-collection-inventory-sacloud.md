---
title: "さくらのクラウド × Ansible: モダンなInventory Pluginで快適インフラ管理"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["さくらのクラウド", "cloud"]
published: false
---

近年ではコンテナをデプロイしてアプリケーションを作ることが多いですが､ansible でプロビジョニングしなくてはいけないケースもままあります｡

ansible でさくらのクラウドの inventory になるモノと言えば､[sakura-internet/sacloud-ansible-inventory](https://github.com/sakura-internet/sacloud-ansible-inventory/) があります｡これは非常に便利なのですが､inventory scripts で実装されています｡

最近の ansible では inventory plugin という仕組みがあります｡これを利用すると ansible-galaxy で管理できる上､設定もわかりやすく記述することができるという利点があります｡

そこで､今回､わたくしは inventory plugins 形式で再発明してみました｡

github においてあるのでご利用ください｡

https://github.com/tokuhirom/ansible-collection-inventory-sacloud

## 使い方

requirements.yml に以下のように記述します｡

```yaml
collections:
  - name: https://github.com/tokuhirom/ansible-collection-inventory-sacloud/releases/download/v0.0.5/tokuhirom-sacloud-0.0.5.tar.gz
```

以下のようにして依存をインストールします｡

```shell
ansible-galaxy collection install -r requirements.yml
```

あとはインベントリファイルを以下のように記述するだけでOKです｡

```yaml
plugin: tokuhirom.sacloud.sacloud
api_token: <your_sacloud_api_token>
api_secret: <your_sacloud_api_secret>
zones:
  - tk1b
```

以下のようにして､動かすことが出来ます｡

```shell
ansible-inventory -i inventory.yml --list -y
```

API token は `SAKURA_ACCESS_TOKEN`, `SAKURA_ACCESS_TOKEN_SECRET` という環境変数で渡すことも可能です｡



