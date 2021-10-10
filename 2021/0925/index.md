---
title: "M1 MacでGatsbyのDocker環境を作成する"
date: "2021-09-25T15:19:03.284Z"
category: "t"
description: ""
emoji: "🐳"
slug: "8543239"
published: true
---

## きっかけ

* M1 Macを使うことになり新たにブログの環境構築する必要があった
* 様々なアーキテクチャのCPUに対応したい（単一作業で）
* ローカルでnode.jsのバージョンを管理したくない（面倒だから）
* 誰でも簡単に立ち上げられるようにしたい

最近はM1 Macを使っているが別のWebアプリを立ち上げる時にCPUアーキテクチャの違いから苦戦することが多かった。
試行錯誤する間にローカル環境が汚されていくのは良い気分がしないから基本的にWeb開発はDockerを使っていくことにした。

## 開発環境

```shell:title=Zsh
% system_profiler SPHardwareDataType
Hardware:

    Hardware Overview:

      Model Name: MacBook Pro
      Model Identifier: MacBookPro17,1
      Chip: Apple M1
      Total Number of Cores: 8 (4 performance and 4 efficiency)
      Memory: 16 GB
      System Firmware Version: 6723.140.2
      OS Loader Version: 6723.140.2

% sw_vers
ProductName:	macOS
ProductVersion:	11.6
BuildVersion:	20G165

% docker -v
Docker version 20.10.8, build 3967b7d

# Dockerコンテナ内
$ gatsby -v
Gatsby CLI version: 3.14.0
Gatsby version: 3.13.0

$ node -v
v16.10.0

$ yarn -v
1.22.5
```

## TL;DR

下記のファイルをプロジェクトルート下に作成。

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
    container_name: blog
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

### コマンド

```shell:title=Zsh
# Dockerfileのイメージを構築
% docker compose build

# 構築したイメージからコンテナの構築、起動（バックグラウンドで）
% docker compose up -d

# コンテナに入る
% docker exec -it {コンテナ名} /bin/bash

# package.jsonに記載されたモジュールをインストール
% yarn

# 開発用サーバの起動
% yarn develop -H 0.0.0.0
```

[[n | Tips]]
| `docker compose`や`docker exec`のような長いコマンドはエイリアスとして登録しておくと便利。
|
| ```shell:title=.zshrc
| alias dc='docker compose'
| alias de='docker exec'
| ```

## 説明

### Dockerfile

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

インストールしているモジュールにもよると思うけど、このブログの場合は上記の構成で上手くいった。
最近リリースされたDebian 11のイメージ`bullseye`の最新版を入れているが今のところ不便なところはない。

一応、`build-essential`を入れることで対応できている。
今後ブログをアップデートする上で必要なパッケージも増えていくと思うのでその際は当記事に追記していく予定。

追記

初めにGatsbyのスターターを使うときはクローンするのでGitが必要。

### その他

ちょっと気になった点としてDockerからGatsbyサーバを起動させる際に`yarn develop`（`gatsby develop`）のみだとlocalhostでアクセスした際に`ERR_EMPTY_RESPONSE`と表示されアクセスできないことだ。
これはホストマシンとコンテナのlocalhostが異なることが原因で発生したエラーである。

解決策としてサーバ起動時にホストを明示的に指定すれば解決する。

```shell:title=Bash
$ yarn develop -H 0.0.0.0
```

`0.0.0.0`でサーバを建てると、そのホストの全てのインターフェースでLISTENする。

ただ、毎回このように書くのは面倒なので`package.json`の`scripts`を書き換えるといいかも。

```diff:title=package.json
"scripts": {
    "build": "git submodule update --remote && gatsby build",
-   "develop": "gatsby develop",
+   "develop": "gatsby develop -H 0.0.0.0",
    "format": "prettier --write \"**/*.{js,jsx,ts,tsx,json,md}\"",
    "start": "yarn run develop",
    "serve": "gatsby serve",
    "clean": "gatsby clean",
    "test": "echo \"Write tests! -> https://gatsby.dev/unit-testing\" && exit 1"
  }
```

[[i | 参考]]
| [docker上のアプリにlocalhostでアクセスしたらERR\_EMPTY\_RESPONSEが出る](https://qiita.com/amuyikam/items/01a8c16e3ddbcc734a46)

## さいごに

### 心配している点

* ~~node-sassだと動かないらしい~~
  * CSS in JSで運用していくから問題なさそう
* 画像をWebP化した時に何かしら問題が出そうな気がする（sharp関連）
