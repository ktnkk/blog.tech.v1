---
title: "リポジトリごとに GitHub アカウントを切り替える"
date: "2021-12-11T20:41:56.421Z"
category: "t"
description: ""
emoji: "🎭"
slug: "8347723"
published: true
---

## モチベーション

プロジェクトごとに GitHub のアカウントを変えたい場合がある。
例えば、実験用や会社用のアカウントなど。
基本的には 1 つのアカウントで全てのバージョン管理するべきではあるが、それができない場合の対処法を備忘録としてまとめる。

ちなみに [GitHub の規約](https://docs.github.com/en/github/site-policy/github-terms-of-service#3-account-requirements) に書いてあるが、1 人の個人または法人が維持できる無料アカウントは 1 つだけである。
そのため無料アカウントを複数持つことはできない。
私は無料のアカウントと PRO アカウントの両方を運用している。

> One person or legal entity may maintain no more than one free Account (if you choose to control a machine account as well, that's fine, but it can only be used for running a machine).

## 開発環境

```shell:title=Zsh {outputLines: 2-9, 11-13, 15}{}
system_profiler SPHardwareDataType
Hardware:

    Hardware Overview:

      Model Name: MacBook Pro
      Model Identifier: MacBookPro17,1
      Chip: Apple M1

sw_vers
ProductName: macOS
ProductVersion: 12.0.1

git --version
git version 2.33.0
```

## 方法 1

まずは GitHub から対象のリポジトリをクローンする。
プロトコルは HTTPS または SSH があるけど、個人的には SSH がおすすめ。
SSH を使う場合は公開鍵認証をしておく必要がある。

また、 SSH をコマンドラインで書く場合はオプションが長くなってしまうため `.ssh/config` にあらかじめオプションを記載しておく。

```text:title=.ssh/ssh_config
Host github_main
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_github_main
    Port 22
    TCPKeepAlive yes
    IdentitiesOnly yes
    AddKeysToAgent yes

Host github_sub
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_github_sub
    Port 22
    TCPKeepAlive yes
    IdentitiesOnly yes
    AddKeysToAgent yes
```

- Host -> 任意の接続名
- HostName -> ホスト名
- User -> ユーザ名
- IdentityFile -> 接続に使用する秘密鍵へのパス
- Port -> ポート番号（ SSH のウェルノウンポートは `22` ）
- TCPKeepAlive -> 通信遮断時に通信先の生存を確認
- IdentitiesOnly -> 指定された秘密鍵のみを使用
- AddKeysToAgent -> 認証に使用した鍵を `ssh-agent` に保存

以下は私のデフォルト設定。
これを読み込んでから `Include` されたファイルに読み込みが移る。

```text:title=.ssh/config
Include ~/.ssh/ssh_config

# LogLevel DEBUG
Compression no
ServerAliveInterval 15
ServerAliveCountMax 3
ExitOnForwardFailure yes
ConnectionAttempts 3
ControlMaster auto
Ciphers aes256-gcm@openssh.com,chacha20-poly1305@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org,ecdh-sha2-nistp521,ecdh-sha2-nistp384,ecdh-sha2-nistp256,diffie-hellman-group18-sha512,diffie-hellman-group16-sha512,diffie-hellman-group14-sha256,diffie-hellman-group-exchange-sha256
Macs umac-128-etm@openssh.com,hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128@openssh.com,hmac-sha2-512,hmac-sha2-256,umac-64-etm@openssh.com,umac-64@openssh.com,hmac-sha1
HostKeyAlgorithms ssh-ed25519-cert-v01@openssh.com,ssh-ed25519,ecdsa-sha2-nistp521-cert-v01@openssh.com,ecdsa-sha2-nistp384-cert-v01@openssh.com,ecdsa-sha2-nistp256-cert-v01@openssh.com,ecdsa-sha2-nistp521,ecdsa-sha2-nistp384,ecdsa-sha2-nistp256,ssh-rsa-cert-v01@openssh.com,rsa-sha2-512,rsa-sha2-256,ssh-rsa
RekeyLimit default 600
VisualHostKey yes
```

```shell:title=Zsh
git clone git@{HOST}:{USER_NAME}/{REPOSITORY_NAME}.git
```

リポジトリをクローンしたらまず最初に下記を設定する。
この作業を忘れるとグローバル（`~/.gitconfig`）に設定したアカウントでコミットされるので注意。

```shell:title=Zsh {outputLines: 1, 5-6, 8-10, 12-13, 15-16, 18-19}{}
// 以下は対象のプロジェクトに移動した後に行う
git config --local user.name "{YOUR_USER_NAME}"
git config --local user.email "{YOUR_EMAIL_ADDRESS}"
git config --local url."{HOST}".insteadOf "git@github.com"

// URL がセットされていなければ
git remote set-url origin git@{HOST}:{YOUR_USER_NAME}/{YOUR_REPOSITORY_NAME}.git

// 以下はオプション
// GPG 署名をオフにしたいとき
git config --local commit.gpgsign false

// GPG署名をオンにしたい時
git config --local commit.gpgsign true

// GPG へのパス
git config --local gpg.program /opt/homebrew/bin/gpg

// GPG の公開鍵
git config --local user.signingkey "{YOUR_GPG_PUB_KEY}"
```

GPG へのパスだが、Homebrew を使った場合のパスを表記している。
なお、 Intel 版は `/usr/local/bin/gpg`、M1 版は `/opt/homebrew/bin/gpg` とパスが違う点に注意が必要。

コマンドを打つのが面倒であれば `-e` オプションでエディタを開いて編集可能。

## 方法 2

上記では SSH の設定でアカウントを分けた。
最近知ったが、 GitHub の設定ファイルを弄ることでより簡単に切り替えができる。

```text:title=.gitconfig
[includeIf "gitdir:~/{YOUR_REPOSITORY_FOLDER}"]
    path = ~/.gitconfig_sub
```

`includeIf` に別アカウントで管理したいリポジトリをまとめたフォルダを指定する。
そして、 `path` 先にそれ専用の `.gitconfig` を書いてあげればプロジェクトごとに設定を書く手間が省ける。

## さいごに

方法 2 がおすすめ。
方法 1 だと毎回設定ファイルを書くのが面倒だし、何より書き忘れたときが大変だ。
結局は 1 つの PC で 1 つのアカウントだけ使うという運用が合理的だな。
