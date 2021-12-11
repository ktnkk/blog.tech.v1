---
title: "GitHub に GPG 署名付きコミットをする"
date: "2021-12-11T19:41:56.421Z"
category: "t"
description: ""
emoji: "🖋"
slug: "3524748"
published: false
---

GitHubサーフィンをしているとまれにコミットに署名（Verified）されたものがある。
これは一体どうなっているのかと調べたらGNU Privacy Guardという暗号化ソフトで作られた鍵を使用したものらしい。
コミットが偽装されることはないと思うが、せっかくなので取り入れてみることにした。
これはその時のメモ。もちろん乱数は変えてある。

## 開発環境

## a

まずはHomebrewでGPGをインストール。

```
brew install gpg
```

次に鍵を作成していく。なんか色々聞かれるけど答えていけばOK。

```
LANG=C gpg --gen-key\ngpg (GnuPG) 2.2.27; Copyright (C) 2021 Free Software Foundation, Inc.\nThis is free software: you are free to change and redistribute it.\nThere is NO WARRANTY, to the extent permitted by law.\n\ngpg: directory &#39;/Users/ki/.gnupg&#39; created\ngpg: keybox &#39;/Users/ki/.gnupg/pubring.kbx&#39; created\nNote: Use &quot;gpg --full-generate-key&quot; for a full featured key generation dialog.\n\nGnuPG needs to construct a user ID to identify your key.\n\nReal name: Keiten Kiki\nEmail address: oo@kiki.ooo\nYou selected this USER-ID:\n    &quot;Keiten Kiki &lt;oo@kiki.ooo&gt;&quot;\n\nChange (N)ame, (E)mail, or (O)kay/(Q)uit? o\nWe need to generate a lot of random bytes. It is a good idea to perform\nsome other action (type on the keyboard, move the mouse, utilize the\ndisks) during the prime generation; this gives the random number\ngenerator a better chance to gain enough entropy.\nWe need to generate a lot of random bytes. It is a good idea to perform\nsome other action (type on the keyboard, move the mouse, utilize the\ndisks) during the prime generation; this gives the random number\ngenerator a better chance to gain enough entropy.\ngpg: /Users/ki/.gnupg/trustdb.gpg: trustdb created\ngpg: key 6GR6R7GPTXE84CVY marked as ultimately trusted\ngpg: directory &#39;/Users/ki/.gnupg/openpgp-revocs.d&#39; created\ngpg: revocation certificate stored as &#39;/Users/ki/.gnupg/openpgp-revocs.d/HW27S6G7472ZC8C8YC4S33Q62FW5KCK2YLDLMGA6.rev&#39;\npublic and secret key created and signed.\n\npub   rsa3072 2021-02-12 [SC] [expires: 2023-02-12]\n      HW27S6G7472ZC8C8YC4S33Q62FW5KCK2YLDLMGA6\nuid                      Keiten Kiki &lt;oo@kiki.ooo&gt;\nsub   rsa3072 2021-02-12 [E] [expires: 2023-02-12]<
```

生成された鍵を確認する。

```
gpg -k\ngpg: 信用データベースの検査\ngpg: marginals needed: 3  completes needed: 1  trust model: pgp\ngpg: 深さ: 0  有効性:   1  署名:   0  信用: 0-, 0q, 0n, 0m, 0f, 1u\ngpg: 次回の信用データベース検査は、2023-02-12です\n/Users/ki/.gnupg/pubring.kbx\n----------------------------\npub   rsa3072 2021-02-12 [SC] [有効期限: 2023-02-12]\n      HW27S6G7472ZC8C8YC4S33Q62FW5KCK2YLDLMGA6\nuid           [  究極  ] Keiten Kiki &lt;oo@kiki.ooo&gt;\nsub   rsa3072 2021-02-12 [E] [有効期限: 2023-02-12]
```

上記のpubキーをコピーしてエクスポート。

```
gpg -a --export HW27S6G7472ZC8C8YC4S33Q62FW5KCK2YLDLMGA6 &gt; pubkey.gpg
```

この段階で鍵作成時に登録した名前とメールアドレスを設定ファイルに記述する。このテキストが間違ってると後で失敗するから注意。

```
git config --global user.name &quot;Keiten Kiki&quot;\n\n% git config --global user.email &quot;oo@kiki.ooo&quot;
```

一応確認。

```
git config --list\ncredential.helper=osxkeychain\nuser.name=Keiten Kiki\nuser.email=oo@kiki.ooo
```

そして、GPGの情報も設定ファイルに追記していく。

```
git config --global gpg.program /usr/local/bin/gpg\n\n% git config --global user.signingkey &quot;HW27S6G7472ZC8C8YC4S33Q62FW5KCK2YLDLMGA6&quot;\n\n% git config --global commit.gpgsign true\n\n% git config --list\ncredential.helper=osxkeychain\nuser.name=Keiten Kiki\nuser.email=oo@kiki.ooo\nuser.signingkey=HW27S6G7472ZC8C8YC4S33Q62FW5KCK2YLDLMGA6\ngpg.program=/usr/local/bin/gpg\ncommit.gpgsign=true
```

ここで試しにコミットしてプッシュするが失敗。

```
git add .\n\n% git commit -m &quot;同じ品番を登録しようとした時のバリデーションを追加&quot;\nerror: gpg failed to sign the data\nfatal: failed to write commit object
```

調べるとMacの場合、pinentryがないとパスフレーズを入れられないらしい。
Homebrewでインストールする。

```
brew install pinentry-mac\n\n% echo &quot;pinentry-program /usr/local/bin/pinentry-mac&quot; &gt;&gt; ~/.gnupg/gpg-agent.conf\n\n% killall gpg-agent
```

これで上手くいった。

```
git commit -m &quot;同じ品番を登録しようとした時のバリデーションを追加&quot;\n[feature-validation eeae61f] 同じ品番を登録しようとした時のバリデーションを追加\n 3 files changed, 184 insertions(+), 180 deletions(-)
```

公開鍵をGitHubに登録するためにクリップボードにコピー。設定からペーストする。

```
cat pubkey.gpg | pbcopy\n
```

```
-----BEGIN PGP PUBLIC KEY BLOCK-----\n\nce*yzcN#8D*^Hz69U$cRVS3#GEyQg4ri4WDrFEdMDctqhg6U7wTiLYfWFgjbph*4\nUn+DTY+TUf/7r0O0CJK0PGE5jBh5nrHWXP0voTSyGG5stI17bZAHDQc56Mr7C6iq\nZS!G$Px!s#vytb$EWx3H&FFJoNzn$z5sHEpCz&5^7VZ89Ty^x2v%p66jiU*jmbBN\n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\n=32A4\n\n\n-----END PGP PUBLIC KEY BLOCK-----
```

こんな感じにペーストされるので保存すれば完了。署名付きのコミットができていればOK。
