---
title: "[WIP] 本を捲るアニメーションを実装する"
date: "2021-10-04T15:36:44.284Z"
category: "t"
description: ""
emoji: "📖"
slug: "5279222"
published: true
---

**※ 執筆途中です。**

## モチベーション

* 実際に本を捲るようなアニメーションを実装してみたい
* Reactのコンポーネントとして手札に持っておきたい

## 開発環境

```shell:title=Zsh {outputLines: 2-13, 15-18, 20-22, 24-26, 28-29, 31}{}
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
ProductName:	macOS
ProductVersion:	11.6
BuildVersion:	20G165

docker -v
Docker version 20.10.8, build 3967b7d

# Dockerコンテナ内
gatsby -v
Gatsby CLI version: 3.14.0
Gatsby version: 3.14.1

node -v
v16.10.0

yarn -v
1.22.5
```

## テスト環境の作成

すぐに捨てられるような実験環境を作っていく。
まずは下記のファイルを空の新規プロジェクトルートに作成。

```dockerfile:title=Dockerfile
FROM node:16.10.0-bullseye-slim
WORKDIR /home/node/app
RUN apt update -q \
    && apt upgrade -y \
    && apt install -y \
       build-essential \
       git
RUN yarn global add gatsby-cli
EXPOSE 8000
```

```yaml:title=docker-compose.yml
version: "3.9"
services:
  dev:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: book
    ports:
      - "8000:8000"
    volumes:
      - .:/home/node/app
    environment:
      - NODE_ENV=development
    tty: true
    stdin_open: true
```

```ignore:title=.dockerignore
node_modules
```

プロジェクトルートに移動後、下記のコマンドを実行。

```shell:title=Zsh {outputLines: 2}{}
docker compose up --build -d

docker exec -it book /bin/bash
```

下記はコンテナ内。

```shell:title=Bash {outputLines: 2, 4, 6}{}
gatsby new book https://github.com/gatsbyjs/gatsby-starter-default

mv book/* book/.[^\.]* . && rm -rf package-lock.json book

yarn

yarn develop -H 0.0.0.0
```

すぐに開発に取り掛かれるようにスターターを使用する。
今回使用したのは`gatsby-starter-default`という最もベーシックなものだ。
多くの人が使っているということはそれだけメンテナンスもされているということなので特に理由がなければいつもこれにしている。

次に作業ディレクトリ下に作られたスターターのファイル類をプロジェクトルートに移動させる。そして、空になったディレクトリを削除する。

ついでに`package-lock.json`も削除して`yarn`でパッケージを管理する手筈を整えて開発サーバを動かせばいつものGatsbyおじさんが現れるはずだ。ここまで10分もかからずに用意できた。

## react-pageflipの導入

### react-pageflipのインストール

ここから本題のページアニメーションの実装に取り掛かっていく。
この分野のモジュールでは[Turn.js](https://github.com/blasten/turn.js/)が有名だが全然メンテナンスがされていない（最後のコミットが2013年5月）し、jQueryは使いたくないので選択肢から外れた。

色々と調べてみて良さそうだと思ったのが今回使用する[react-pageflip](https://github.com/Nodlik/react-pageflip)である。
これは[StPageFlip](https://github.com/Nodlik/StPageFlip)がReact用に改良されたものである（同じ作者）。

早速、インストールしてみる。

```shell:title=Bash {outputLines: 2, 4}{}
yarn add react-pageflip

yarn info react-pageflip version
2.0.3
```

### styled-componentsのインストール

ついでに後でスタイリングする際に使うstyled-componentsもインストールする（任意）。

```shell:title=Bash {outputLines: 2, 4-5, 7-8, 10}{}
yarn add styled-components gatsby-plugin-styled-components babel-plugin-styled-components

yarn info styled-components version
5.3.1

yarn info gatsby-plugin-styled-components version
4.14.0

yarn info babel-plugin-styled-components version
1.13.2
```

```diff:title=gatsby-config.js
module.exports = {
  siteMetadata: {"title": `Gatsby Default Starter`...},
  plugins: [
+ `gatsby-plugin-styled-components`
  ]
}
```

## react-pageflipで遊ぶ

gatsby-starter-defaultは便利だがスタイルの書き方や無駄なページがあったりと最初に綺麗にする手間がどうにも面倒くさい。
そういったニーズに合わせたスターターもあるにはあるが、ちょっと古かったりするので結局使わずにここまで来た。
適当に綺麗にしたと仮定して、公式ページのマニュアルに沿ってハンズオンで進めていく。

### 基本的な使い方

コンポーネントとして使わない場合はimport後にそのまま使える。


