---
title: "anyenv と nodenv を導入する"
date: "2021-11-13T15:31:56.421Z"
category: "t"
description: ""
emoji: "📦"
slug: "2725272"
published: true
---

## モチベーション

ここ最近は全ての開発環境を Docker コンテナで管理していた。
これはローカル環境を汚したくないからというのが大きな理由である（今思えば謎のこだわり）。
コマンド 1 つで簡単に起動できるし、特に問題なく運用できていた。

しかし、最近は TypeScript を書く機会が徐々に増えてきて入力補完やリンターに頼りたくなってきた。
現在は IntelliJ IDEA という IDE を使用しているのだが、なぜかインタプリタを Docker に接続できない。

もちろん、[公式ドキュメント](https://www.jetbrains.com/help/idea/node-with-docker.html)も読んだ。
だけど、上手くいかないし、あまり環境構築に時間を掛けたくないので妥協してローカルでバージョン管理をすることにした。

今回導入したのは Node.js を管理するための[nodenv](https://github.com/nodenv/nodenv)と`**env`系のパッケージマネージャを管理する[anyenv](https://github.com/anyenv/anyenv)である。
後々、Ruby のパッケージマネージャである[rbenv](https://github.com/rbenv/rbenv)も入れる予定。

導入方法自体は README を見れば書いてあるが、ちょっとしたメモとして書き留めておく。

## 開発環境

```shell:title=Zsh {outputLines: 2-13, 15-18, 20-21, 23-24, 26}{}
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

brew -v
Homebrew 3.3.3

anyenv -v
anyenv 1.1.4

nodenv -v
nodenv 1.4.0+3.631d0b6
```

## 導入手順

### anyenv

```shell:title=Zsh
brew install anyenv
anyenv init
echo 'eval "$(anyenv init -)"' >> ~/.zshrc
```

#### 主要コマンド

| コマンド               | 内容                                             |
| ---------------------- | ------------------------------------------------ |
| `anyenv install -l`    | インストールできる\*\*env の一覧                 |
| `anyenv install **env` | 指定された\*\*env のインストール                 |
| `anyenv versions`      | \*\*env ごとのインストールされている実行環境一覧 |
| `anyenv version`       | \*\*env ごとのシェルにおける実行環境             |
| `anyenv update`        | 一括アップデート（`anyenv-update`を使用）        |

---

### nodenv

```shell:title=Zsh
anyenv install nodenv
exec $SHELL -l
```

#### 主要コマンド

| コマンド                     | 内容                                        |
| ---------------------------- | ------------------------------------------- |
| `nodenv install -l`          | インストールできるバージョンの一覧          |
| `nodenv install {version}`   | 指定されたバージョンのインストール          |
| `nodenv uninstall {version}` | 指定されたバージョンのアンインストール      |
| `nodenv global {version}`    | グローバルで使うバージョンを指定する        |
| `nodenv local {version}`     | ローカルで使うバージョンを指定する          |
| `anyenv versions`            | インストールされている Node.js 実行環境一覧 |
| `anyenv version`             | シェルにおける Node.js 実行環境             |

---

### anyenv-update

anyenv のプラグイン。
これを取り入れることで`anyenv update`コマンドを使用できる。

```shell:title=Zsh
mkdir -p $(anyenv root)/plugins
git clone https://github.com/znz/anyenv-update.git $(anyenv root)/plugins/anyenv-update
```

---

### nodenv default-package

nodenv のプラグイン。
Node.js をインストールする際、自動的に指定したパッケージをインストールできる。

```shell:title=Zsh
git clone https://github.com/nodenv/nodenv-default-packages.git "$(nodenv root)/plugins/nodenv-default-packages"
touch $(nodenv root)/default-packages
echo "yarn\ntypescript\nts-node\ntypesync" >> $(nodenv root)/default-packages
```

```text:title=./.anyenv/envs/nodenv/default-packages
yarn
typescript
ts-node
typesync
```

## さいごに

素晴らしいエコシステム。
突然 IDE が怒りっぽくなったけど、修正オプションを提示してくれるのでコーディングが捗る。
あとは、コンテナ上で実行するよりも当然だがローカルの方が断然早い。
これからは基本的にバージョン管理は`**env`を使う予定だけど、誰でも簡単に環境を構築できるという点で Docker には勝らないので合わせて開発していきたい。
