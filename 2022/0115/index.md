---
title: "[WIP] 最強の Gatsby テンプレートを作る"
date: "2022-01-15T23:19:03.284Z"
category: "t"
description: ""
emoji: "🏗"
slug: "7282726"
published: true
---

## モチベーション

ここ数年でフロントエンドの分野は大きく発展した。
その発展のスピードが早すぎて最新技術のキャッチアップに追われる日々である。

それが良いことなのか悪いことなのかはさておき、現在は皆さんご存じのとおり npm (yarn) と呼ばれるパッケージマネージャで親切な人たちが開発した OSS を一括管理する方法が主流である。
それらのパッケージを組み合わせて Web アプリを作っていくことになるのだが、開発環境を構築する際にルーティーンのようにパッケージをインストールしていかなければならない。

これが非常に面倒くさい。プログラマたるもの面倒なことをなくすために面倒なことをすることは厭わないが、ドキュメントを見てマニュアルどおりに作業するのは完全に時間の無駄である。

だから、ある程度使い込んで知見を得たフレームワークについては今後テンプレート化してすぐに開発に取り掛かれるような体制を作っていこうと思う。

GitHub の Public template という機能を使うとそのままコピーしてテンプレートを使えるのでプライベートや仕事で活用できそうだ。

### テンプレート化の予定

一応、以下のフレームワーク・ライブラリのテンプレート化を予定している。

- Gatsby.js
- Next.js
- Google Apps Scrips (Clasp)
- Ruby on Rails
- Go
- Shopify theme
- Shopify app (Ruby on Rails, Node.js)

いずれもよく使うものなのですぐに使えるメリットはでかい。
そもそも、今思えば新しく使うフレームワークでもテンプレートは同時に作って運用しても良さそうな気がする。
使っていく上で改善点があれば反映させれば良いし（ログも残る）、汎用的な環境構築を意識する面でも効果的だ。
せっかく作るのなら他の人にも使って欲しいから最強なテンプレートを作る。

### 最強の定義

エンジニア界隈では強いというワードがよく見受けられる。
強いエンジニアといえば達人といった意味である（英語圏では Wizard や Guru みたいな）。
それに倣って最強なテンプレートを作っていくのが目標であるが、最強の定義とはなんだろう。
少なくとも以下のキーワードは最低限クリアしたい。

1. インスタント
2. 安全
3. 自動化
4. 文書化

まず最も大事なのはすぐに使えるということである。
逆にすぐに使えなければテンプレートを導入する意味がない。
また、設定ファイルが好きで過剰に設定を盛り込みすぎる癖があるから属人的にならない設定にしたい（デフォルト設定でも良さそう）。

次に大切なのは安全かどうかである。
型安全性は JS の代わりに TS を使うとかで対応するがそこは大した問題ではない。
いくつものライブラリを組み合わせていくと必ず脆弱性が見つかったりして放置していると大変なことになってしまう。
Apache Log4j に深刻な脆弱性が見つかったのも記憶に新しい。
不審なライブラリは使わないこととすぐに依存関係のアップデートをできる仕組み作りが大事である。

3 つ目に大事なのは面倒な作業は自動化させるということだ。
ツール大好き人間なので便利そうなライブラリは初期段階で全部詰め込んでしまう。
しかし、それらを手動で毎回動かすとなると結構な時間的コストが発生してしまう。
それらのツールは適宜トリガーを仕掛けることで自動的に動くようにする。
もちろん初めからちゃんとコードを書けていれば必要ないことではあるが、人間はミスをするという前提の元で考えている（チーム開発ならなおさら）。

最後 4 つ目はきっちり文書化することである。
誰でも使いやすいように README を書くことは面倒だが大事だ。
面倒くさがり & 飽きっぽい性格なのでコードを書いたらあとは放置してしまい、後から見て何をしたかったのか分からないパターンがよくある。
README は他人に理解してもらうためのドキュメントでもあるが、その前に自分が理解できるものでなければならない。
[Standard Readme](https://github.com/RichardLitt/standard-readme) というスタイルが汎用的で良さそうなので、キャッチアップしてから取り入れようと思う。

以上の項目をテンプレート作成の指針としていく。

## Gatsby.js

今回は React 向けの静的サイトジェネレータ（SSG）[Gatsby.js](https://github.com/gatsbyjs/gatsby) 向けのテンプレートを作る（SSR もできるが使ったことはない）。
2022 年現在、ポートフォリオサイトやブログを構築するのであれば最高の選択肢だと思っている。
それゆえにプロジェクトを作る機会も多いから真っ先にテンプレートを作ることにした。

https://github.com/ktnkk/gatsby-starter

↑ がリポジトリ。

## リンターとフォーマッター

リンター（静的解析）とフォーマッターには下記のツールを使用している。

| 名称                                                               | 用途                               | 設定ファイル           |
| ------------------------------------------------------------------ | ---------------------------------- | ---------------------- |
| [ESLint](https://github.com/eslint/eslint)                         | JS（TS）のための静的検証ツール     | `.eslintrc.json`       |
| [Prettier](https://github.com/prettier/prettier)                   | ソースコードの整形                 | `.prettierrc`          |
| [EditorConfig](https://github.com/editorconfig/editorconfig)       | エディタ用のフォーマットルール設定 | `.editorconfig`        |
| [commitlint](https://github.com/conventional-changelog/commitlint) | コミット規約に準じているかチェック | `.commitlintrc.json`   |
| [Markdownlint](https://github.com/markdownlint/markdownlint)       | マークダウン構文を解析             | `.markdownlintrc.json` |
| [textlint](https://github.com/textlint/textlint)                   | テキストファイルを解析             | `.textlintrc.json`     |

基本的にはデフォルトの設定しか入れていない。
プロジェクトの特性に合わせて設定を変えてもいいし、そのままでも十分使える。
むしろ、修行僧でもなければデフォルトでも十分な気がする。
commitlint では [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) というコミット規則に基づいたコミットがされているかをチェックしている。
Conventional Commits は [Angular](https://github.com/angular/angular.js) のコミット規則として作られたものである。
prefix や scope を記載することでコミット履歴を参照しやすいし、後述する semantic-release にも活用できる。

```text:title=Text
fix(firebase): add submodule token to merge config
```

上記のような感じ。
ただ、これを毎回手打ちするのは面倒なので [Commitizen](https://github.com/commitizen/cz-cli) というライブラリを使う。
これを入れておけば `yarn cz` でプログラムが立ち上がり対話的に Conventional Commits を作成できる。

## pre-commit

リンターやフォーマッターは IDE やエディタと連携させることで効果を最大限に引き出せる。
だが、警告に気付かない場合や無視したくなるときがあるので安全のためにコミット前検証をする。

| 名称                                                 | 用途                                                     | 設定ファイル         |
| ---------------------------------------------------- | -------------------------------------------------------- | -------------------- |
| [Husky](https://github.com/typicode/husky)           | `git commit` や `git push` 時にコマンドを実行            | `.husky/*`           |
| [lint-staged](https://github.com/okonet/lint-staged) | ステージングしたファイルに対してのみコマンドを実行できる | `.lintstagedrc.json` |

上記のツールを使うことでコミット前には必ずリンターとフォーマッター、テストを通せる。

## タグとリリース

GitHub にはタグとリリース機能がある（[このような感じ](https://github.com/ktnkk/gatsby-starter/releases)）。
これは CLI または GitHub 上で追加できるのだが、かなり面倒くさい。
この面倒くささが原因でやる気が削がれてはもったいないのでこれも自動化する。
その際に [semantic-release](https://github.com/semantic-release/semantic-release) というツールを使う。
これは前述した Conventional Commits に基づいてバージョンを管理してくれるので非常に便利だ。
バージョン自体は GitHub 推奨の [Semantic Versioning 2.0.0](https://semver.org/) がデフォルト設定である。
例えば以下のような prefix がバージョンが上がるトリガーになる。

- fix: パッチバージョン
  - 1.0.0 -> 1.0.1
- feat: マイナーバージョン
  - 1.0.0 -> 1.1.0
- BREAKING CHANGE（footer に記載）-> メジャーバージョン
  - 1.0.0 -> 2.0.0

リリース自体は main ブランチにマージされた時点で GitHub Actions が行ってくれる。

```yaml:title=release.yml
name: Release
on:
  push:
    branches:
      - main
jobs:
  release:
    runs-on: ubuntu-20.04
    steps:
      - name: Checkout this repository
        uses: actions/checkout@v2
      - name: Setup node
        uses: actions/setup-node@v1
        with:
          node-version: "16.13.0"
      - name: Install dependencies
        run: yarn --frozen-lockfile
      - name: Release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: yarn semantic-release
```

## その他諸々のツール

| 名称                                     | 用途                     | 設定ファイル |
| ---------------------------------------- | ------------------------ | ------------ |
| [Hygen](https://github.com/jondot/hygen) | 対話式コードジェネレータ | `.hygen.js`  |
| [Docker](https://www.docker.com/)        | 開発環境構築             | `Dockerfile` |

このテンプレートでは Atomic Design に基づいてコンポーネントを作成するようにしている。
もちろん別のデザインフレームワークでも対応できるが。
ただ、Atomic Design だと細かくコンポーネントを分けるため必然的にファイル量が多くなってしまう（Storybook や スタイルシートなど）。
ある程度中身のテンプレートが決まっているのであれば Hygen を使って動的に生成した方が効率的である。

Docker も Web 開発には欠かせない。
個人的には単一のコンテナしか立ち上げないのであればローカルで環境を整えたほうが良いとは思う（[nodenv](https://github.com/nodenv/nodenv) みたいな便利なパッケージマネージャもあるし）。
どうしても重くはなるし、データベースやプロキシを組み合わせるのでなければ旨味はないかな。
一応入れてはみたけど。

## GitHub Actions

| 名称                                                        | 用途                                          |
| ----------------------------------------------------------- | --------------------------------------------- |
| [Dependabot](https://github.com/dependabot/dependabot-core) | 依存ライブラリのアップデートを検知、PR の作成 |
| [CodeQL](https://codeql.github.com/)                        | コードベース全体の脆弱性を検知                |

これらの Actions は cron で定期的に実行される設定にしてある。
Dependabot は非常に便利で、依存関係のアップデートがあれば自動的にプルリクを作ってくれる。
ただし、npm の依存関係は多岐にわたるため放置しておくと相当数プルリクが溜まってしまうことが多い。
セキュリティ面から考えてもバージョンは常に最新にしておくべきだから仕方がないとはいえ手間だからどうにかしたいところだ。
また、リポジトリのセキュリティの項目にある Dependabot Alerts は Active にするのを忘れないようにしたい。

## テスト

前述した ESLint と Prettier はソースコードをパースするだけで実行はしない。
そこでテストランナーとして [Jest](https://github.com/facebook/jest) を使用する。
Jest は JSDOM 経由で DOM をシミュレートする。
DOM のレイヤーが抽象化されているため、 React の描画結果を Node.js でテストが可能になる。
導入段階になって問題が発生したのでテンプレートには現段階では入れていない。
ローカルでテストをテストして問題なければ後日マージする予定である。

また、リグレッションテストには [Storybook](https://github.com/storybookjs/storybook) の StoryShots を使うがこれも一時保留。
Storybook は UI コンポーネントのカタログとしても活用できるので必ず入れた方が良い。
GitHub Actions と連携させて GitHub Pages で Storybook をホスティングする方法を取り入れたいと思っている。

他には下記のテストツールを入れる予定。

- インタラクションテスト: [Enzyme](https://github.com/enzymejs/enzyme)
- レイアウトテスト: [BackstopJS](https://github.com/garris/BackstopJS)
- アクセシビリティテスト: [axe](https://github.com/dequelabs/axe-core)

正直なところ、テストは考え方も含めて完璧に理解できていない分野なのである程度学習してからテンプレートに取り入れる。

## Gatsby の TypeScript 化

GraphQL の型生成には [gatsby-plugin-typegen](https://github.com/cometkim/gatsby-plugin-typegen) を使用している。
また、設定ファイルの TS 化は [esbuild-register](https://github.com/egoist/esbuild-register) を使って esbuild で TS をトランスパイルしている。
しかし、`gatsby-config.js` がエントリーポイントになるため、このファイルだけは残念ながら TS 化できない。
今後不具合があれば戻すけど、現時点では特に問題ない。
個人的には [ts-node](https://github.com/TypeStrong/ts-node) を使うよりも高速なのでおすすめ（その代わり型チェックされないなどのデメリットはある）。

## 最低限入れるべきプラグイン

Gatsby はプラグインを入れることで WordPress みたいに簡単に機能を拡張できる。
今までサイトを作ってきた中で必ず入れた方が良いプラグインだけ厳選してみた。

Gatsby でサイトを作るということは Web 上に公開する前提だと思うので最低限の SEO 対策をする。

| プラグイン名                                                                                         | 用途                                |
| ---------------------------------------------------------------------------------------------------- | ----------------------------------- |
| [gatsby-plugin-sitemap](https://www.gatsbyjs.com/plugins/gatsby-plugin-sitemap/)                     | サイトマップの自動生成              |
| [gatsby-plugin-robots-txt](https://www.gatsbyjs.com/plugins/gatsby-plugin-robots-txt/)               | `robot.txt` の自動生成              |
| [gatsby-plugin-react-helmet](https://www.gatsbyjs.com/plugins/gatsby-plugin-react-helmet/)           | meta タグの設定に必要               |
| [gatsby-plugin-google-tagmanager](https://www.gatsbyjs.com/plugins/gatsby-plugin-google-tagmanager/) | Google タグマネージャとの連携に使用 |

データ分析をみっちりやろうと思ったらタグマネージャは必須なのでアナリティクスの代わりに入れている。
ちょっと面倒だけどタグマネージャの管理画面からアナリティクスとの連携作業が必要になる。
下記はテンプレートに入れるかどうか迷っているプラグイン。

| プラグイン名                                                                                       | 用途                             |
| -------------------------------------------------------------------------------------------------- | -------------------------------- |
| [gatsby-plugin-nprogress](https://www.gatsbyjs.com/plugins/gatsby-plugin-nprogress/)               | プログレスバーを表示             |
| [gatsby-plugin-feed](https://www.gatsbyjs.com/plugins/gatsby-plugin-feed/)                         | RSS フィードを作成               |
| [gatsby-plugin-canonical-urls](https://www.gatsbyjs.com/plugins/gatsby-plugin-canonical-urls/)     | URL 正規化                       |
| [gatsby-plugin-manifest](https://www.gatsbyjs.com/plugins/gatsby-plugin-manifest/)                 | アプリを PWA 対応にする          |
| [gatsby-remark-autolink-headers](https://www.gatsbyjs.com/plugins/gatsby-remark-autolink-headers/) | タグにリンクをつける             |
| [gatsby-plugin-offline](https://www.gatsbyjs.com/plugins/gatsby-plugin-offline/)                   | ビルド時にサービスワーカーを生成 |

これらのプラグインは必ずしも必要ではない。
画像処理系のプラグイン（`sharp` 系）はけっこうトラブルを起こすからテンプレートを複製してから入れても良さそう。

## さいごに

テンプレートを作りながら思ったが、圧倒的に SEO とテストに関する知識が足りていない。
サイトの成長には必須の知識だから早いことマスターしたい。
タイトルに WIP（work in progress）と書いたようにこのテンプレートはまだ未完成だから改善点があったら随時アップデートしていく。
