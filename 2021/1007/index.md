---
title: "Docker を使った Shopify CLI の環境構築"
date: "2021-10-07T15:36:44.284Z"
category: "t"
description: ""
emoji: "🛍"
slug: "3543892"
published: true
---

## プロローグ

今の時代はメルカリやヤフオクなど誰でも簡単に要らなくなったものを適正な価格で売買できる。
しかし、よく考えてみると手数料も高いし、何だか損してるなと感じるのは贅沢な悩みだろうか。

できることなら自分のショップを独立した形で持てれば色々と捗るだろうから、その分野ではもっとも勢いのある Shopify でストアを作ってみることにした。
あわよくば、それで仕事の幅を広げられたら良いな。

まずはお店の外観となるテーマの編集をしていきたい。[Shopify CLI](https://github.com/Shopify/shopify-cli)という便利ツールを使っていく。
正直なところ、そんなに売るものもないから形だけでもしっかりしたものを作れれば良いな。

## 開発環境

今回も Docker を使って開発環境を整えていく。ただ、最後の最後に詰まったということは初めに書いておきたい。
私の知識不足が大きな原因だが、そんなに拘りがないのであれば Docker は使わないほうが賢明である。
まだバグが多いし、けっこう仕様の変更がある。

```shell:title=Zsh {outputLines: 2-13, 15-18, 20-22, 24-25, 27-28, 30}{}
system_profiler SPHardwareDataType
Hardware:

    Hardware Overview:

      Model Name: MacBook Pro
      Model Identifier: MacBookPro17,1
      Chip: Apple M1
      Total Number of Cores: 8 (4 performance and 4 efficiency)
      Memory: 16 GB
      System Firmware Version: 6723.140.2
      OS Loader Version: 6723.140.2

sw_vers
ProductName: macOS
ProductVersion: 11.6
BuildVersion: 20G165

docker -v
Docker version 20.10.8, build 3967b7d

# Dockerコンテナ内
ruby -v
ruby 3.0.2p107 (2021-07-07 revision 0db68f0233) [aarch64-linux]

node -v
v16.10.0

shopify version
2.6.3
```

## 作業手順

空のプロジェクトを作成して以下のファイルから Docker コンテナを生成する。

```dockerfile:title=Dockerfile
FROM ruby:3.0.2-bullseye

RUN apt update -q \
    && apt upgrade -y \
    && apt install wget sudo -y \
    && gem install shopify-cli -v 2.6.3 \
    && mkdir -p ~/.config/shopify \
    && printf "[analytics]\nenabled = false\n" > ~/.config/shopify/config \
    && curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash - \
    && sudo apt install -y nodejs \
       vim

WORKDIR /usr/src/app
```

8 行目の`printf "[analytics]\nenabled = false\n" > ~/.config/shopify/config`は何を意味するのか。
それは shopify-cli のコマンド`shopify config analytics --disable`と同じことである。

これを書くことで匿名の使用状況レポートを送信されないようにする。どんなに匿名と言っても信用できない。

後からファイル編集をする必要があるので`vim`も入れておく。

```yaml:title=docker-compose.yml
version: "3.9"
services:
  dev:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: shopify
    ports:
      - "3456:3456"
      - "9292:9292"
    volumes:
      - .:/usr/src/app
    tty: true
    stdin_open: true
```

ポートフォワーディングに関しては`3456`は認証時に使用、`9292`はプレビューを見るときに使用するので開けておく。
今回はテーマしか編集しないけど Rails や Node.js を使ったアプリ開発をするのであれば対応したポートを開ける必要がある。

```shell:title=Zsh {outputLines: 2}{}
docker compose up --build -d

docker exec -it shopify /bin/bash
```

これでコンテナを起動し、中に入ることができた。

```shell:title=Bash {outputLines: 2-5, 7-8, 10-12}{}
shopify login --store ktnkk.myshopify.com
Please open this URL in your browser:
https://accounts.shopify.com/oauth/authorize?client_id=fbdb2649-e327-4907-8f67-908d24cfd7e3&scope=openid+https%3A%2F%2Fapi.shopify.com%2Fauth%2Fshop.admin.graphql+https%3A%2F%2Fapi.shopify.com%2Fauth%2Fshop.admin.themes+https%3A%2F%2Fapi.shopify.com%2Fauth%2Fpartners.collaborator-relationships.readonly+https%3A%2F%2Fapi.shopify.com%2Fauth%2Fshop.storefront-renderer.devtools+https%3A%2F%2Fapi.shopify.com%2Fauth%2Fpartners.app.cli.access&redirect_uri=http%3A%2F%2F127.0.0.1%3A3456&state=31fdcb5f41373a5b88770a1a7fa9553d5a9a001ad63076c7cfa98aefae2f&response_type=code&code_challenge=Qow2mbHXW6S9uAip2FHb0VcwOKBm9DPQ0VXlT8K94lE&code_challenge_method=S256
Logged into store ktnkk.myshopify.com in partner organization ktnkk

shopify whoami
Logged into store ktnkk.myshopify.com in partner organization ktnkk

shopify theme init Dawn
┏━━ Cloning https://github.com/Shopify/dawn.git into Dawn… ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
┃                                                                                                                                                                                                                                         100%
┗━━ ✓ Cloned into Dawn ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ (3.28s) ━━
```

早速ストア名を使って OAuth 認証する。
URL が発行されるのでブラウザからアクセスすれば OK。

`shopify whoami`でログインしているパートナー組織、またはスタッフメンバーとしてログインしているストアを表示する。

テーマはもっともベーシックな Dawn を入れてみた。以下のような構造をしている。

```shell:title=Bash {outputLines: 2, 4-20}{}
apt install tree

tree -L 2
.
├── Dawn
│   ├── LICENSE.md
│   ├── README.md
│   ├── assets
│   ├── config
│   ├── layout
│   ├── locales
│   ├── sections
│   ├── snippets
│   ├── templates
│   └── translation.yml
├── Dockerfile
├── docker-compose.yml
├── node_modules
├── shopify.iml
└── yarn.lock
```

GitHub を使ってテーマを管理するのであればプロジェクトルートにダウンロードしたディレクトリを配置しなければならない（上記の例では`Dawn`ディレクトリ内に配置されてしまっている）。

```shell:title=Bash {outputLines: 2-4, 6-8}{}
shopify theme serve
✗ You are not authorized to edit themes on ktnkk.myshopify.com.
Make sure you are a user of that store, and allowed to edit themes.

shopify login
Please open this URL in your browser:
https://accounts.shopify.com/oauth/authorize?client_id=fbdb2649-e327-4907-8f67-908d24cfd7e3&scope=openid+https%3A%2F%2Fapi.shopify.com%2Fauth%2Fshop.admin.graphql+https%3A%2F%2Fapi.shopify.com%2Fauth%2Fshop.admin.themes+https%3A%2F%2Fapi.shopify.com%2Fauth%2Fpartners.collaborator-relationships.readonly+https%3A%2F%2Fapi.shopify.com%2Fauth%2Fshop.storefront-renderer.devtools+https%3A%2F%2Fapi.shopify.com%2Fauth%2Fpartners.app.cli.access&redirect_uri=http%3A%2F%2F127.0.0.1%3A3456&state=a1d24c729e5ae816a7aa085b32a669717b375b25137caf18bcfe950b494d&response_type=code&code_challenge=hQNSpzIq0lehHkRYvW3nfEdLeflSbCGDkPuX3gh2bmQ&code_challenge_method=S256
Logged into store ktnkk.myshopify.com in partner organization ktnkk
```

開発サーバを立ち上げてみる。しかし、失敗。さっき認証したはずなのに何故か認証されていないらしい。仕方なくもう一度ログイン（ストア名は要らない）するとこの問題は解決した。

```shell:title=Bash {outputLines: 2-18}{}
shopify theme serve
┏━━ Viewing theme… ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
┃ * Syncing theme #127861457150 on ktnkk.myshopify.com
┃                                                                                                                                                                                                                                         100%
┃
┃ Serving .
┃
┃ Please open this URL in your browser:
┃ http://127.0.0.1:9292
┃
┃ Customize this theme in the Online Store Editor:
┃ https://ktnkk.myshopify.com/admin/themes/127861457150/editor
┃
┃ Share this theme preview:
┃ https://ktnkk.myshopify.com/?preview_theme_id=127861457150
┃
┃ (Use Ctrl-C to stop)
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ (108.93s) ━━
```

ここからが詰まりポイント。
`shopify theme serve`で開発サーバを立ち上げてみると成功した風な表示が出る。

しかし、ブラウザ（残念ながら Google Chrome しか対応していない）からアクセスしてみると`ERR_EMPTY_RESPONSE`と表示されてプレビューが表示されない。

この原因はコンテナ外からリクエストが来ているのに開発サーバが`localhost`（`http://127.0.0.1`）で Listen してしまっているからである。
なのでホストが持つ全てのインタフェースで Listen できるように`0.0.0.0`でサーバを建てる必要がある。

ただ、残念なことに Shopify CLI にはオプションでホスト IP を設定できないようだ。
Gatsby であれば`gatsby develop -H 0.0.0.0`、Rails であれば`rails server -b 0.0.0.0`みたいに書けるイメージだったのだが。

遠回りだが今回インストールした Gem（shopify-cli）を直接書き換えるしかない。
以下がその diff。

```shell:title=Bash {outputLines: 2}{}
cd /usr/local/bundle/gems/shopify-cli-2.6.3/lib/shopify_cli/theme/dev_server

vi web_server.rb
```

Web サーバの設定をしている[Ruby ファイル](https://github.com/Shopify/shopify-cli/blob/0701e2820b74fe58e9df6ebcb322fb0f63bf39bc/lib/shopify_cli/theme/dev_server/web_server.rb#L31)を修正。

```diff:title=web_server.rb
# Base on Rack::Handler::WEBrick
class WebServer < ::WEBrick::HTTPServlet::AbstractServlet
  def self.run(app, **options)
    environment  = ENV["RACK_ENV"] || "development"
-   default_host = environment == "development" ? "localhost" : nil
+   default_host = environment == "development" ? "0.0.0.0" : nil

    if !options[:BindAddress] || options[:Host]
      options[:BindAddress] = options.delete(:Host) || default_host
    end
    options[:Port] ||= 8080
    if options[:SSLEnable]
      require "webrick/https"
    end
```

これだけではプレビューを変更したときに投げられる HTTP リクエストでトークンに関するエラーが出るので WEBrick（Ruby の Web サーバライブラリ）の[設定](https://github.com/Shopify/shopify-cli/blob/6e8521863045721a6714660b6784378fd8e907a8/vendor/deps/webrick/lib/webrick/httprequest.rb#L601)を弄る。

```shell:title=Bash {outputLines: 2}{}
cd /usr/local/bundle/gems/shopify-cli-2.6.3/vendor/deps/webrick/lib/webrick

vi httprequest.rb
```

```diff:title=httprequest.rb
PrivateNetworkRegexp = /
  ^unknown$|
- ^((::ffff:)?127.0.0.1|::1)$|
+ ^((::ffff:)?0.0.0.0|::1)$|
  ^(::ffff:)?(10|172\.(1[6-9]|2[0-9]|3[01])|192\.168)\.
/ixo
```

これでようやくプレビュー画面を表示させることができた。
ホットリロードに対応しているため即座にデザインを確かめられる点が素晴らしい。

Shopify について調べて実際にストアを構築する中で開発面での充実度がかなり高いなと感じている。
[Liquid](https://github.com/Shopify/liquid)という奇妙なテンプレート言語もあるし、API も豊富で極めればすごく面白いことができそうだ。

## あとがき

Docker での環境構築をするには現時点で Gem 内部を弄る必要があるのであまりオススメしない。
どうしてもしたいのであれば止めないが何かあっても自己責任でお願いしたい。

ただ、流石に Docker コンテナを再構築するたびにファイルを変更するのは面倒なので Shopify CLI のリポジトリに[Issue](https://github.com/Shopify/shopify-cli/issues/1615)を投げてみた。
せめてオプションが指定できれば良いのだがどうかな。
