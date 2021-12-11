---
title: "SSH で GitHub に接続する手順"
date: "2021-12-11T18:41:56.421Z"
category: "t"
description: ""
emoji: "🔒"
slug: "5294467"
published: false
---

新しいPCを使うことになり当然GitHubに接続するのだが、意外と忘れていたので今後のためにメモしておく。
作業自体は簡単だけどそんなに頻繁にすることでもないから無理に覚えて脳内ディスクを消費する必要もない。
ちなみにこれらの鍵はダミーなので不正アクセスを試みて貴重な時間を無駄にしないように。

## 開発環境

## a

まずはSSHキーの置いてある隠しフォルダに移動する。

```
cd ~/.ssh\ncd: no such file or directory: /Users/ki/.ssh
```

が、そもそもデフォルトでは存在しないのフォルダなので仮のSSHキーを生成してみる。

```shell
ssh-keygen\nGenerating public/private rsa key pair.\nEnter file in which to save the key (/Users/ki/.ssh/id_rsa):\nCreated directory &#39;/Users/ki/.ssh&#39;.\nEnter passphrase (empty for no passphrase):\nEnter same passphrase again:\nYour identification has been saved in /Users/ki/.ssh/id_rsa.\nYour public key has been saved in /Users/ki/.ssh/id_rsa.pub.\nThe key fingerprint is:\nSHA256:dAKV6XDV7vhnRYHxVTTtoQbFmFXdgjtU+UUZOMhfoVo ki@my.local\nThe key&#39;s randomart image is:\n+---[RSA 3072]----+\n|      ...+o.B**O&|\n|      ..+  *+===B|\n|       +o .ooE+=+|\n|       ..o  *+  +|\n|        S  +.. . |\n|          . .   .|\n|           .   . |\n|            . o  |\n|             o   |\n+----[SHA256]-----+
```

秘密鍵と公開鍵が生成された。<code>ls</code>コマンドで確認してみる。

```shell
ls -al ~/.ssh\ntotal 16\ndrwx------   4 ki  staff   128  2 11 22:29 .\ndrwxr-xr-x+ 28 ki  staff   896  2 11 22:29 ..\n-rw-------   1 ki  staff  2635  2 11 22:29 id_rsa\n-rw-r--r--   1 ki  staff   565  2 11 22:29 id_rsa.pub
```

すると、デフォルトでは鍵長が2635と少し物足りない。せっかくなら長いほうが良いのでオプションを指定して生成する。オプションの中身を見れば大体分かると思う。

```shell
ssh-keygen -t rsa -b 4096 -C &quot;oo@kiki.ooo&quot; -f ~/.ssh/id_rsa_github_private\nGenerating public/private rsa key pair.\nCreated directory &#39;/Users/ki/.ssh&#39;.\nEnter passphrase (empty for no passphrase):\nEnter same passphrase again:\nYour identification has been saved in /Users/ki/.ssh/id_rsa_github_private.\nYour public key has been saved in /Users/ki/.ssh/id_rsa_github_private.pub.\nThe key fingerprint is:\nSHA256:Nz5kdxTXWPSburf4STKM3h29FVW86ezjTZ2WZVTRsKB+c oo@kiki.ooo\nThe key&#39;s randomart image is:\n+---[RSA 4096]----+\n|           =+...=|\n|            +o.o.|\n|   .        .o.. |\n|  + .         +. |\n| o . .  S    . ..|\n|    . .. .  .   .|\n| . o o .....    .|\n|+ = * . .... o=.+|\n|oOE+.o   .o .+**B|\n+----[SHA256]-----+\n\n% ssh-keygen -l -f ~/.ssh/id_rsa_github_private.pub\n4096 SHA256:Nz5kdxTXWPSburf4STKM3h29FVW86ezjTZ2WZVTRsKB+c oo@kiki.ooo (RSA)
```

生成された秘密鍵をGitHubにコピペするためにクリップボードにコピーする。

```shell
pbcopy &lt; ~/.ssh/id_rsa_github_private
```

GitHubにペーストした後にSSHで接続できるか検証。なぜか、接続失敗。

```shell
ssh -T git@github.com\nThe authenticity of host &#39;github.com (52.69.186.44)&#39; can&#39;t be established.\nRSA key fingerprint is SHA256:Nz5kdxTXWPSburf4STKM3h29FVW86ezjTZ2WZVTRsKB.\nAre you sure you want to continue connecting (yes/no/[fingerprint])? yes\nWarning: Permanently added &#39;github.com,52.69.186.44&#39; (RSA) to the list of known hosts.\ngit@github.com: Permission denied (publickey).
```

それもそのはずで設定を書いてなかった。

```.ssh/config
Host github_p\nHostName github.com\nUser git\nIdentityFile ~/.ssh/id_rsa_github_private\nPort 22\nTCPKeepAlive yes\nIdentitiesOnly yes\nAddKeysToAgent yes</code></pre>
```

改めてSSHで接続（ログ付き）してみる。

```
ssh -vT github_p\nOpenSSH_8.1p1, LibreSSL 2.7.3\ndebug1: Reading configuration data /Users/ki/.ssh/config\ndebug1: /Users/ki/.ssh/config line 1: Applying options for github_p\ndebug1: Reading configuration data /etc/ssh/ssh_config\ndebug1: /etc/ssh/ssh_config line 47: Applying options for *\ndebug1: Connecting to github.com port 22.\ndebug1: Connection established.\ndebug1: identity file /Users/ki/.ssh/id_rsa_github_private type 0\ndebug1: identity file /Users/ki/.ssh/id_rsa_github_private-cert type -1\ndebug1: Local version string SSH-2.0-OpenSSH_8.1\ndebug1: Remote protocol version 2.0, remote software version babeld-c34a939f\ndebug1: no match: babeld-c34a939f\ndebug1: Authenticating to github.com:22 as &#39;git&#39;\ndebug1: SSH2_MSG_KEXINIT sent\ndebug1: SSH2_MSG_KEXINIT received\ndebug1: kex: algorithm: curve25519-sha256\ndebug1: kex: host key algorithm: rsa-sha2-512\ndebug1: kex: server-&gt;client cipher: chacha20-poly1305@openssh.com MAC: &lt;implicit&gt; compression: none\ndebug1: kex: client-&gt;server cipher: chacha20-poly1305@openssh.com MAC: &lt;implicit&gt; compression: none\ndebug1: expecting SSH2_MSG_KEX_ECDH_REPLY\ndebug1: Server host key: ssh-rsa SHA256:Nz5kdxTXWPSburf4STKM3h29FVW86ezjTZ2WZVTRsKB\ndebug1: Host &#39;github.com&#39; is known and matches the RSA host key.\ndebug1: Found key in /Users/ki/.ssh/known_hosts:1\nWarning: Permanently added the RSA host key for IP address &#39;52.192.72.89&#39; to the list of known hosts.\ndebug1: rekey out after 134217728 blocks\ndebug1: SSH2_MSG_NEWKEYS sent\ndebug1: expecting SSH2_MSG_NEWKEYS\ndebug1: SSH2_MSG_NEWKEYS received\ndebug1: rekey in after 134217728 blocks\ndebug1: Will attempt key: /Users/ki/.ssh/id_rsa_github_private RSA SHA256:6n7Z+Vc5AgkN/bVuS6goCVBdoq34L26v/65yf7YVxYw explicit\ndebug1: SSH2_MSG_EXT_INFO received\ndebug1: kex_input_ext_info: server-sig-algs=&lt;ssh-ed25519-cert-v01@openssh.com,ecdsa-sha2-nistp521-cert-v01@openssh.com,ecdsa-sha2-nistp384-cert-v01@openssh.com,ecdsa-sha2-nistp256-cert-v01@openssh.com,rsa-sha2-512-cert-v01@openssh.com,rsa-sha2-256-cert-v01@openssh.com,ssh-rsa-cert-v01@openssh.com,ssh-dss-cert-v01@openssh.com,ssh-ed25519,ecdsa-sha2-nistp521,ecdsa-sha2-nistp384,ecdsa-sha2-nistp256,rsa-sha2-512,rsa-sha2-256,ssh-rsa,ssh-dss&gt;\ndebug1: SSH2_MSG_SERVICE_ACCEPT received\ndebug1: Authentications that can continue: publickey\ndebug1: Next authentication method: publickey\ndebug1: Offering public key: /Users/ki/.ssh/id_rsa_github_private RSA SHA256:6n7Z+Vc5AgkN/bVuS6goCVBdoq34L26v/65yf7YVxYw explicit\ndebug1: Server accepts key: /Users/ki/.ssh/id_rsa_github_private RSA SHA256:6n7Z+Vc5AgkN/bVuS6goCVBdoq34L26v/65yf7YVxYw explicit\nEnter passphrase for key &#39;/Users/ki/.ssh/id_rsa_github_private&#39;:\ndebug1: Authentication succeeded (publickey).\nAuthenticated to github.com ([52.192.72.89]:22).\ndebug1: channel 0: new [client-session]\ndebug1: Entering interactive session.\ndebug1: pledge: network\ndebug1: Sending environment.\ndebug1: Sending env LC_TERMINAL_VERSION = 3.4.4\ndebug1: Sending env LANG = ja_JP.UTF-8\ndebug1: Sending env LC_TERMINAL = iTerm2\ndebug1: client_input_channel_req: channel 0 rtype exit-status reply 0\nHi ktnkk! You&#39;ve successfully authenticated, but GitHub does not provide shell access.\ndebug1: channel 0: free: client-session, nchannels 1\nTransferred: sent 3564, received 2724 bytes, in 0.4 seconds\nBytes per second: sent 9029.9, received 6901.6\ndebug1: Exit status 1
```

成功！
