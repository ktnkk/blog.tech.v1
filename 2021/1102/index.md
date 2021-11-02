---
title: "textlintで校正を自動化する"
date: "2021-11-02T18:08:56.421Z"
category: "t"
description: ""
emoji: "🧐"
slug: "3445436"
published: true
---

<!-- textlint-disable -->

## モチベーション

記事を書いているとどうしても自分の文章が気になってしまう。 やっぱり正しい日本語を使いたいと思っているし、それが読みやすさにも繋がるはずだ。
しかし、何度も読み返すほど暇ではないからどうしたものかと調べてみると[textlint](https://textlint.github.io/)というテキストのためのリンターがあったので早速導入してみることにした。

## 開発環境

```shell:title=Bash {outputLines: 2-3, 5-6, 8}{}
node -v
v17.0.1

yarn -v
1.22.15

yarn info textlint version
12.0.2
```

## 導入

```shell:title=Baxh {outputLines: 2, 4}{}
yarn add -D textlint

touch .textlintrc.js

touch .textlintignore
```

1. 開発環境でしか使用しないので`-D`オプションをつけてインストールする
2. 設定ファイル`.textlintrc.js`を作成する
3. 無視したいファイルがあれば`.textlintignore`を作成して記載する。

設定ファイルの拡張子は`yaml`, `js`, `json`からお好みで。
これで準備は整ったので[マニュアル](https://github.com/textlint/textlint/wiki/Collection-of-textlint-rule#rules-japanese)を見ながら、当ブログで使えそうなルールを設定していく。

## マニフェスト

* 1文の長さは100文字以下
* セクションの最初の1文の長さは50文字以下
* 「、」は1文中に3つまで
* 連続できる最大の漢字長は6文字まで
* 「ですます調」、「である調」を統一する
* 文末の句点記号として「」を使う
* 二重否定は使用しない
* ら抜き言葉を使用しない
* 逆接の接続助詞「が」を連続して使用しない
* 同じ接続詞を連続して使用しない
* 同じ助詞を連続して使用しない
* 半角カナを使用しない
* 弱い日本語表現を使用しない
* よくある日本語の誤用をチェックする
* 冗長な表現をチェックする
* 入力ミスで発生する不自然なアルファベットをチェックする
* 漢字よりもひらがなで表記すべき形式名詞、副詞、補助動詞をチェックする
* 例示・並列表現の「～たり、（～たり）する」をチェックする
* サ抜き、サ入れ表現の誤用をチェックする
* 「ええと」「あの」「まあ」などのフィラー（つなぎ表現）を禁止する
* ドキュメント内のすべてのリンクが使用可能であることを確認する
* 漢数字と算用数字を使い分ける

## 書き方ルールを設定

`.textlintrc.js`に記載してあるオプションは当ブログで採用したものである。

### [textlint-rule-max-ten](https://github.com/textlint-ja/textlint-rule-max-ten)

一文に利用できる`、`の数を制限できる。一文の読点の数が多いと冗長で読みにくい文章となるため、読点の数を一定数以下にしたい場合は有効。

```shell:title=Bash
yarn add -D textlint-rule-max-ten
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "max-ten": {
      // 1文に利用できる最大の、の数
      max: 3,
      // 例外ルールを適応するかどうか
      strict: false,
      // 読点として扱う文字
      touten: "、",
      // 句点として扱う文字
      kuten: "。"
    }
  }
};
```

***

### [textlint-rule-max-kanji-continuous-len](https://github.com/textlint-ja/textlint-rule-max-kanji-continuous-len)

漢字が連続する最大文字数を制限できる。

```shell:title=Bash
yarn add -D textlint-rule-max-kanji-continuous-len
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "max-kanji-continuous-len": {
      // 連続できる漢字の文字数
      max: 6,
      // 例外として無視する単語
      allow: ["東急田園都市線"]
    }
  }
};
```

***

### [textlint-rule-no-mix-dearu-desumasu](https://github.com/textlint-ja/textlint-rule-no-mix-dearu-desumasu)

敬体（ですます調）と常体（である調）の混在をチェックする。

```shell:title=Bash
yarn add -D textlint-rule-no-mix-dearu-desumasu
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "no-mix-dearu-desumasu": {
      // 見出し
      preferInHeader: "である",
      // 本文
      preferInBody: "である",
      // 箇条書き
      preferInList: "である",
      // 文末以外でも、敬体と常体を厳しくチェックするかどうか
      strict: true
    }
  }
}
```

***

### [textlint-rule-no-doubled-joshi](https://github.com/textlint-ja/textlint-rule-no-doubled-joshi)

1つの文中に同じ助詞が連続して出てくるのをチェックする。

```shell:title=Bash
yarn add -D textlint-rule-no-doubled-joshi
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "no-doubled-joshi": {
      // 助詞のtoken同士の間隔値が1以下ならエラーにする
      min_interval: 1,
      // 例外を許可するかどうか
      strict: false,
      // 複数回の出現を許す助詞
      allow: ["も", "や"],
      // 文の区切り文字となる配列
      separatorCharacters: ["。", "？", "！"],
      commaCharacters: ["、"]
    }
  }
}
```

***

### [textlint-rule-no-double-negative-ja](https://github.com/textlint-ja/textlint-rule-no-double-negative-ja)

二重否定を検出する。

```shell:title=Bash
yarn add -D textlint-rule-no-double-negative-ja
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "no-double-negative-ja" : true
  }
}
```

***

### [textlint-rule-no-hankaku-kana](https://github.com/azu/textlint-rule-no-hankaku-kana)

半角カナの利用を禁止する。

```shell
yarn add -D textlint-rule-no-hankaku-kana
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "no-hankaku-kana": true
  }
}
```

***

### [textlint-rule-ja-no-weak-phrase](https://github.com/textlint-ja/textlint-rule-ja-no-weak-phrase)

弱い日本語表現の利用を禁止する。

* 〜かもしれない
* 〜と思う
* 〜と思います
* 可能性を示唆する

上記のような言葉が弱い日本語に該当する。

```shell:title=Bash
yarn add -D textlint-rule-ja-no-weak-phrase
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "ja-no-weak-phrase": true
  }
}
```

***

### [textlint-rule-ja-no-redundant-expression](https://github.com/textlint-ja/textlint-rule-ja-no-redundant-expression)

冗長な表現（その文から省いても意味が通じるような表現）を禁止する。

```shell:title=Bash
yarn add -D textlint-rule-ja-no-redundant-expression
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "ja-no-redundant-expression": true
  }
}
```

***

### [textlint-rule-ja-no-abusage](https://github.com/textlint-ja/textlint-rule-ja-no-abusage)

よくある誤用をチェックする（下記は辞書の定義）。

* [dict/prh.yml](https://github.com/textlint-ja/textlint-rule-ja-no-abusage/blob/master/dict/prh.yml)
* [src/dictionary.ts](https://github.com/textlint-ja/textlint-rule-ja-no-abusage/blob/master/src/dictionary.ts)

```shell:title=Bash
yarn add -D textlint-rule-ja-no-abusage
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "ja-no-abusage": true
  }
}
```

***

### [textlint-rule-no-mixed-zenkaku-and-hankaku-alphabet](https://github.com/textlint-ja/textlint-rule-no-mixed-zenkaku-and-hankaku-alphabet)

全角と半角アルファベットを混在をチェックする。

```shell:title=Bash
yarn add -D textlint-rule-no-mixed-zenkaku-and-hankaku-alphabet
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "no-mixed-zenkaku-and-hankaku-alphabet": {
      prefer: "半角"
    }
  }
}
```

***

### [textlint-rule-sentence-length](https://github.com/azu/textlint-rule-sentence-length)

センテンスの文字数をチェックする。

```shell:title=Bash
yarn add -D textlint-rule-sentence-length
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "sentence-length": {
      max: 100,
      skipUrlStringLink: true
    }
  }
}
```

***

### [textlint-rule-first-sentence-length](https://github.com/azu/textlint-rule-first-sentence-length)

セクションの最初のセンテンスの文字数をチェックする。

```shell:title=Bash
yarn add -D textlint-rule-first-sentence-length
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "first-sentence-length": {
      "max": 50
    }
  }
}
```

***

### [textlint-rule-no-dropping-the-ra](https://github.com/textlint-ja/textlint-rule-no-dropping-the-ra)

ら抜き言葉を検出する。

```shell:title=Bash
yarn add -D textlint-rule-no-dropping-the-ra
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "no-dropping-the-ra": true
  }
}
```

***

### [textlint-rule-no-doubled-conjunctive-particle-ga](https://github.com/takahashim/textlint-rule-no-doubled-conjunctive-particle-ga)

逆接の接続助詞「が」は、特に否定の意味ではなくても安易に使われてしまいがちである。
これが同一文中に複数回出現していないかどうかをチェックする。

```shell:title=Bash
yarn add -D textlint-rule-no-doubled-conjunctive-particle-ga
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "no-doubled-conjunctive-particle-ga": true
  }
}
```

***

### [textlint-rule-no-doubled-conjunction](https://github.com/takahashim/textlint-rule-no-doubled-conjunction)

同じ接続詞が連続して出現していないかどうかをチェックする。

```shell:title=Bash
yarn add -D textlint-rule-no-doubled-conjunction
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "no-doubled-conjunction": true
  }
}
```

***

### [textlint-rule-ja-no-mixed-period](https://github.com/textlint-ja/textlint-rule-ja-no-mixed-period)

文末の句点（）の統一or抜けをチェックする。

```shell:title=Bash
yarn add -D textlint-rule-ja-no-mixed-period
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "ja-no-mixed-period": {
      // 優先する句点文字
      periodMark: "。",
      // 句点文字として許可する文字列の配列
      allowPeriodMarks: [],
      // 末尾に絵文字を置くことを許可するか
      allowEmojiAtEnd: false,
      // 句点で終わって無い場合に`periodMark`を--fix時に追加するかどうか
      forceAppendPeriod: false
    }
  }
}
```

***

### [textlint-rule-ja-hiragana-keishikimeishi](https://github.com/lostandfound/textlint-rule-ja-hiragana-keishikimeishi)

漢字よりもひらがなで表記したほうが読みやすい形式名詞を指摘する。自動修正対応。

```shell:title=Bash
yarn add -D textlint-rule-ja-hiragana-keishikimeishi
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "ja-hiragana-keishikimeishi": true
  }
}
```

***

### [textlint-rule-ja-hiragana-fukushi](https://github.com/lostandfound/textlint-rule-ja-hiragana-fukushi)

漢字よりもひらがなで表記したほうが読みやすい副詞を指摘する。自動修正対応。

```shell:title=Bash
yarn add -D textlint-rule-ja-hiragana-fukushi
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "ja-hiragana-fukushi": true
  }
}
```

***

### [textlint-rule-ja-hiragana-hojodoushi](https://github.com/lostandfound/textlint-rule-ja-hiragana-hojodoushi)

漢字よりもひらがなで表記したほうが読みやすい補助動詞を指摘する。自動修正対応。

```shell:title=Bash
yarn add -D textlint-rule-ja-hiragana-hojodoushi
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "ja-hiragana-hojodoushi": true
  }
}
```

***

### [textlint-rule-ja-unnatural-alphabet](https://github.com/textlint-ja/textlint-rule-ja-unnatural-alphabet)

不自然なアルファベットを検知する。

```shell:title=Bash
yarn add -D textlint-rule-ja-unnatural-alphabet
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "ja-unnatural-alphabet": true
  }
}
```

***

### [textlint-rule-no-insert-dropping-sa](https://github.com/textlint-ja/textlint-rule-no-insert-dropping-sa)

サ抜き、サ入れ表現の誤用をチェックする。自動修正対応。

```shell:title=Bash
yarn add -D @textlint-ja/textlint-rule-no-insert-dropping-sa
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "@textlint-ja/textlint-rule-no-insert-dropping-sa": true
  }
}
```

***

### [textlint-rule-prefer-tari-tari](https://github.com/textlint-ja/textlint-rule-prefer-tari-tari)

例示・並列表現の「～たり、（～たり）する」をチェックする。

```shell:title=Bash
yarn add -D textlint-rule-prefer-tari-tari
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "prefer-tari-tari": true
  }
}
```

### [textlint-rule-no-filler](https://github.com/textlint-ja/textlint-rule-no-filler)

「ええと」「あの」「まあ」などのフィラー（つなぎ表現）を禁止する。

```shell:title=Bash
yarn add -D @textlint-ja/textlint-rule-no-filler
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "@textlint-ja/no-filler": true
  }
}
```

***

### [textlint-rule-no-dead-link](https://github.com/nodaguti/textlint-rule-no-dead-link)

ドキュメント内のすべてのリンクが使用可能であることを確認する。

```shell:title=Bash
yarn add -D textlint-rule-no-dead-link
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "no-dead-link": {
      // 相対URIの可用性をチェック
      checkRelative: true,
      // 相対URIを解決するために使用されるベースURI
      baseURI: "https://ktnkk.com/",
      // 無視するURIまたはglobの配列
      ignore: ["https://github.com/ktnkk/*"],
      // デフォルトのリクエストの代わりにオリジンのURLに接続できるようにするOriginの配列
      preferGET: [],
      // リダイレクト（3xxステータスコード）をチェック
      ignoreRedirects: false,
      // 再試行してURLをチェックする回数
      retry: 3,
      // httpヘッダーをカスタマイズ
      userAgent: "textlint-rule-no-dead-link/1.0"
    }
  }
}
```

***

### [textlint-rule-prh](https://github.com/textlint-rule/textlint-rule-prh)

[prh](https://github.com/prh/prh)を使った校正ヘルパー。
自分で自動修正可能な単語帳を作成できる。

```shell:title=Bash
yarn add -D textlint-rule-prh
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    prh: {
      checkLink: false,
      checkBlockQuote: false,
      checkEmphasis: true,
      checkHeader: false,
      rulePaths: ["textlint/prh.yml"]
    }
  }
}
```

ルールを管理するためのファイルを作成（`textlint/prh.yml`）。

```yaml:title=prh.yml
version: 1

imports:
  - tech.yml
```

上記のファイルに単語帳を作成しても良いが、カテゴリ別にしたかったので別ファイルに切り出した。

```yaml:title=tech.yml
version: 1
rules:
  - expected: jQuery
    specs:
      - from: jquery
        to: jQuery
      - from: ＪＱＵＥＲＹ
        to: jQuery
```

***

### [textlint-rule-preset-JTF-style](https://github.com/textlint-ja/textlint-rule-preset-JTF-style)

数量を表現し、数を数えられるものは算用数字を使用する。
慣用的表現、熟語、概数、固有名詞、副詞など、漢数字を使用することが一般的な語句では漢数字を使う。

```shell:title=Bash
yarn add -D textlint-rule-preset-jtf-style
```

```javascript:title=.textlintrc
module.exports = {
  rules: {
    "preset-jtf-style": true
  }
}
```

### [textlint-plugin-markdown](https://github.com/textlint/textlint/tree/master/packages/@textlint/textlint-plugin-markdown)

Markdown記法を考慮した校正をする。
後から気づいたが、これは組み込みプラグインなのでインストールする必要はなかった。
もし`mdx`を使っている場合は拡張のためにインストールする必要がある。

```shell:title=Bash
yarn add -D @textlint/textlint-plugin-markdown
```

```javascript:title=.textlintrc.js
module.exports = {
  rules: {
    "@textlint/textlint-plugin-markdown": true
  }
}
```

***

### [textlint-filter-rule-comments](https://github.com/textlint/textlint-filter-rule-comments)

コメントディレクティブを使用してエラーを無視する。

```shell:title=Bash
yarn add -D textlint-filter-rule-comments
```

```javascript:title=.textlintrc.js
module.exports = {
  filters: {
    comments: {
      enablingComment: "textlint-enable",
      disablingComment: "textlint-disable"
    }
  }
}
```

下記のようにコメントで挟んで使用する。
特定のルールのみを適用できる。

```markdown:title=Markdown
<!-- textlint-disable no-mix-dearu-desumasu -->
敬体（ですます調）と常体（である調）の混在をチェックする。
<!-- textlint-enable -->
```

***

### [textlint-filter-rule-allowlist](https://github.com/textlint/textlint-filter-rule-allowlist)

リストで許可することによって任意の単語をフィルタリングする。

```shell:title=Bash
yarn add -D textlint-filter-rule-allowlist
```

```javascript:title=.textlintrc.js
module.exports = {
  filters: {
    allowlist: {
      allowlistConfigPaths: ["textlint/allow.yml"]
    }
  }
}
```

許可する単語をリスト化するためのファイル（`textlint/allow.yml`）を作成する。
下記では数式（`KaTeX`）を許可。

```yaml:title=allow.yml
- "/\\$\\$[\\s\\S]*?\\$\\$/m"
```

## 完成した設定ファイル

```javascript:title=.textlintrc.js
module.exports = {
  filters: {
    comments: {
      enablingComment: "textlint-enable",
      disablingComment: "textlint-disable"
    },
    allowlist: {
      allowlistConfigPaths: ["textlint/allow.yml"]
    }
  },
  rules: {
    "max-ten": {
      // 1文に利用できる最大の、の数
      max: 3,
      // 例外ルールを適応するかどうか
      strict: false,
      // 読点として扱う文字
      touten: "、",
      // 句点として扱う文字
      kuten: "。"
    },
    "max-kanji-continuous-len": {
      // 連続できる漢字の文字数
      max: 6,
      // 例外として無視する単語
      allow: ["東急田園都市線"]
    },
    "no-mix-dearu-desumasu": {
      // 見出し
      preferInHeader: "である",
      // 本文
      preferInBody: "である",
      // 箇条書き
      preferInList: "である",
      // 文末以外でも、敬体と常体を厳しくチェックするかどうか
      strict: true
    },
    "no-doubled-joshi": {
      // 助詞のtoken同士の間隔値が1以下ならエラーにする
      min_interval: 1,
      // 例外を許可するかどうか
      strict: false,
      // 複数回の出現を許す助詞
      allow: ["も", "や"],
      // 文の区切り文字となる配列
      separatorCharacters: ["。", "？", "！"],
      commaCharacters: ["、"]
    },
    "no-mixed-zenkaku-and-hankaku-alphabet": {
      prefer: "半角"
    },
    "sentence-length": {
      max: 100,
      skipUrlStringLink: true
    },
    "first-sentence-length": {
      max: 50
    },
    "ja-no-mixed-period": {
      // 優先する句点文字
      periodMark: "。",
      // 句点文字として許可する文字列の配列
      allowPeriodMarks: [],
      // 末尾に絵文字を置くことを許可するか
      allowEmojiAtEnd: false,
      // 句点で終わって無い場合に`periodMark`を--fix時に追加するかどうか
      forceAppendPeriod: false
    },
    "no-dead-link": {
      // 相対URIの可用性をチェック
      checkRelative: true,
      // 相対URIを解決するために使用されるベースURI
      baseURI: "https://ktnkk.com/",
      // 無視するURIまたはglobの配列
      ignore: ["https://github.com/ktnkk/*"],
      // デフォルトのリクエストの代わりにオリジンのURLに接続できるようにするOriginの配列
      preferGET: [],
      // リダイレクト（3xxステータスコード）をチェック
      ignoreRedirects: false,
      // 再試行してURLをチェックする回数
      retry: 3,
      // httpヘッダーをカスタマイズ
      userAgent: "textlint-rule-no-dead-link/1.0"
    },
    prh: {
      checkLink: false,
      checkBlockQuote: false,
      checkEmphasis: true,
      checkHeader: false,
      rulePaths: ["textlint/prh.yml"]
    },
    "no-double-negative-ja": true,
    "no-hankaku-kana": true,
    "ja-no-weak-phrase": true,
    "ja-no-redundant-expression": true,
    "ja-no-abusage": true,
    "no-dropping-the-ra": true,
    "no-doubled-conjunctive-particle-ga": true,
    "no-doubled-conjunction": true,
    "ja-hiragana-keishikimeishi": true,
    "ja-hiragana-fukushi": true,
    "ja-hiragana-hojodoushi": true,
    "ja-unnatural-alphabet": true,
    "@textlint-ja/textlint-rule-no-insert-dropping-sa": true,
    "prefer-tari-tari": true,
    "@textlint-ja/no-filler": true,
    "@textlint/textlint-plugin-markdown": true
  }
};
```

## 実行結果

試しに1つの記事をリンターに解析させてみる。
実行環境はDockerなのだが、想像以上にメモリを消費してしまい`Killed`されて実行できなかった。
メモリの上限を上げたら問題なく実行できた。

```shell:title=Bash {outputLines: 2-42}{}
yarn textlint "content/blog/tech/2021/1009/index.md"

yarn run v1.22.15
$ /home/node/app/node_modules/.bin/textlint content/blog/tech/2021/1009/index.md

/home/node/app/content/blog/tech/2021/1009/index.md
   11:1   error    Line 11 exceeds the maximum sentence length of 50.
First sentence should be short                         first-sentence-length
   19:12  ✓ error  ひらがなで表記したほうが読みやすい形式名詞: 上 => うえ                     ja-hiragana-keishikimeishi
   20:33  ✓ error  ひらがなで表記したほうが読みやすい副詞: "概ね" => "おおむね"               ja-hiragana-fukushi
   28:51  ✓ error  ひらがなで表記したほうが読みやすい副詞: "丁度" => "ちょうど"               ja-hiragana-fukushi
   31:1   error    Line 31 sentence length(176) exceeds the maximum sentence length of 100.
Over 76 characters   sentence-length
   41:13  error    文中に逆接の接続助詞 "が" が二回以上使われています。                       no-doubled-conjunctive-particle-ga
   58:16  error    一文に二回以上利用されている助詞 "は" がみつかりました。                   no-doubled-joshi
   64:17  ✓ error  ひらがなで表記したほうが読みやすい形式名詞: 時 => とき                     ja-hiragana-keishikimeishi
   67:6   ✓ error  ひらがなで表記したほうが読みやすい形式名詞: 時 => とき                     ja-hiragana-keishikimeishi
   67:11  ✓ error  ひらがなで表記したほうが読みやすい形式名詞: 時 => とき                     ja-hiragana-keishikimeishi
   75:13  error    一文に二回以上利用されている助詞 "が" がみつかりました。                   no-doubled-joshi
   86:28  error    弱い表現: "思う" が使われています。                                        ja-no-weak-phrase
   89:11  error    一文に二回以上利用されている助詞 "は" がみつかりました。                   no-doubled-joshi
   90:23  error    一文に二回以上利用されている助詞 "が" がみつかりました。                   no-doubled-joshi
   94:24  ✓ error  ひらがなで表記したほうが読みやすい副詞: "丁度" => "ちょうど"               ja-hiragana-fukushi
   94:29  error    弱い表現: "思う" が使われています。                                        ja-no-weak-phrase
  104:24  ✓ error  ひらがなで表記したほうが読みやすい形式名詞: 時 => とき                     ja-hiragana-keishikimeishi
  104:34  error    弱い表現: "思う" が使われています。                                        ja-no-weak-phrase
  112:16  error    弱い表現: "思う" が使われています。                                        ja-no-weak-phrase
  113:41  error    一文に二回以上利用されている助詞 "に" がみつかりました。                   no-doubled-joshi
  127:11  ✓ error  ひらがなで表記したほうが読みやすい副詞: "最も" => "もっとも"               ja-hiragana-fukushi
  127:34  error    弱い表現: "思う" が使われています。                                        ja-no-weak-phrase
  185:38  error    弱い表現: "思う" が使われています。                                        ja-no-weak-phrase
  220:14  ✓ error  ひらがなで表記したほうが読みやすい形式名詞: 時 => とき                     ja-hiragana-keishikimeishi
  230:33  error    一文に二回以上利用されている助詞 "で" がみつかりました。                   no-doubled-joshi
  249:11  ✓ error  ひらがなで表記したほうが読みやすい副詞: "寧ろ" => "むしろ"                 ja-hiragana-fukushi
  252:25  error    一文に二回以上利用されている助詞 "が" がみつかりました。                   no-doubled-joshi
  259:1   error    Line 259 sentence length(152) exceeds the maximum sentence length of 100.
Over 52 characters  sentence-length
  265:25  error    文末が"。"で終わっていません。                                             ja-no-mixed-period

✖ 29 problems (29 errors, 0 warnings)
✓ 11 fixable problems.
Try to run: $ textlint --fix [file]
```

想像していたよりも大量のエラーでショックを受けた。
1つの記事でこれだから先が思いやられる。
弱い表現を使っていることが多いとのことで、逃げの姿勢が校正によって顕になった。

自動修正可能な表現があるのでコマンドを実行してみる。

```shell:title=Bash {outputLines: 2-19}{}
yarn textlint --fix "content/blog/tech/2021/1009/index.md"
yarn run v1.22.15
$ /home/node/app/node_modules/.bin/textlint --fix content/blog/tech/2021/1009/index.md

/home/node/app/content/blog/tech/2021/1009/index.md
   19:12  ✔   ひらがなで表記したほうが読みやすい形式名詞: 上 => うえ        ja-hiragana-keishikimeishi
   64:17  ✔   ひらがなで表記したほうが読みやすい形式名詞: 時 => とき        ja-hiragana-keishikimeishi
   67:6   ✔   ひらがなで表記したほうが読みやすい形式名詞: 時 => とき        ja-hiragana-keishikimeishi
   67:11  ✔   ひらがなで表記したほうが読みやすい形式名詞: 時 => とき        ja-hiragana-keishikimeishi
  104:24  ✔   ひらがなで表記したほうが読みやすい形式名詞: 時 => とき        ja-hiragana-keishikimeishi
  220:14  ✔   ひらがなで表記したほうが読みやすい形式名詞: 時 => とき        ja-hiragana-keishikimeishi
   20:33  ✔   ひらがなで表記したほうが読みやすい副詞: "概ね" => "おおむね"  ja-hiragana-fukushi
   28:51  ✔   ひらがなで表記したほうが読みやすい副詞: "丁度" => "ちょうど"  ja-hiragana-fukushi
   94:24  ✔   ひらがなで表記したほうが読みやすい副詞: "丁度" => "ちょうど"  ja-hiragana-fukushi
  127:11  ✔   ひらがなで表記したほうが読みやすい副詞: "最も" => "もっとも"  ja-hiragana-fukushi
  249:11  ✔   ひらがなで表記したほうが読みやすい副詞: "寧ろ" => "むしろ"    ja-hiragana-fukushi

✔ Fixed 11 problems
✖ Remaining 1 problem
```

いい感じに修正された。言われてみれば、「むしろ」の読みは分かりにくいよな。
この要領でコマンドを実行しつつ、1つ1つエラーをしらみ潰しにあたる。

他にオプションとして`--rule`を付けることで特定のルールのみ調べることもできる。

```shell:title=Bash
yarn textlint --rule ja-hiragana-hojodoushi README.md
```

## キャッシュで高速化

設定したオプションや記事の量にもよるが、`textlint`の実行は時間がかかる。
少しでも早くするためにキャッシュを使用する。

```shell:title=Bash
yarn textlint --cache "**/*.md"
```

`--cache`オプションを使用することで`.textlintcache`が作成される。
これはGitで管理すべきファイルではないから、`.gitignore`に追加する。

```diff:title=.gitignore
+  # textlint
+  .textlintcache
```

<!-- textlint-enable -->

## さいごに

こんなに修正が大変なら初めからリンターを導入しておけば良かったと後悔した。
これはtextlintだけではなく他のリンターにも当てはまるだろう。できるだけ早く理想の設定ファイルを書いておきたい。

あと、エディターにはIntelliJ IDEAを使用しているのだが、プラグインが対応していないため毎回コマンドを打つのはけっこう面倒くさい。
そもそもフロントだけ書くのであればVSCodeかVimでも良いかなと考え始めた。まだ結論は出そうにないけれど。

今回は全てのサブモジュールに共通した設定を書いたが、カテゴリの種類に応じて設定ファイルを書き換えるのも面白そう。
ただ、サブモジュールにあまり機能を持たせるのはどうかと考えていて導入はしなかった。気が変わる可能性もある。

いずれにせよ、良い文章を書くという目的は忘れないようにしたい。
リンターを使うと自然言語でも人工言語を扱っているような感覚になって面白いのでオススメ。
ちょっとガチガチに規則を縛りすぎたきらいがあるから窮屈に感じるようであれば緩和しよう。

[[i | 関連リンク]]
| * [Issue](https://github.com/ktnkk/blog/issues/104)
| * [Pull request](https://github.com/ktnkk/blog/pull/182)
| * [Techの校正結果](https://github.com/ktnkk/blog/commit/547e0b7da05e52d48fde529f432a0bed20432d62)
| * [Onsenの校正結果](https://github.com/ktnkk/blog/commit/eabe119869cb2b8d3de1b844817fb28a8c5b9fb8)
| * [Lifeの校正結果](https://github.com/ktnkk/blog/commit/6d6284a0bf4426943162dc6e0c89aff628afa4ea)
