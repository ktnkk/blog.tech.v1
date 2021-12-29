---
title: "ktnkk.log の技術スタックとロードマップ"
date: "2021-10-10T13:53:28.000Z"
category: "t"
description: ""
emoji: "🗺"
slug: "3878283"
published: true
---

以前は WordPress を使ってブログ運営をしていた。
しかし、様々な面倒に直面したのと DX が良くない（書く気にならない）ので勉強も兼ねて 1 からブログを作っていくことにした。
隙間時間を見つけてはコードを試すのを繰り返し、大体 4 週間ほどで公開できる段階まで来た。

以下リンクが管理しているリポジトリ。Public なのでどんどんプルリクを出して頂けるとありがたい。

<https://github.com/ktnkk/blog>

## 前回の反省

![前のブログ](01.jpg)

[WordPress](https://github.com/WordPress/WordPress) は素晴らしい CMS であり、批判をするつもりは更々ないけど個人的には下記の項目がとてもストレスであった。

- 記事を書くためにブラウザを立ち上げて Web エディタまで移動しなければならない
- WYSIWYG で出力された HTML の汎用性の無さ
- 使わない機能が多いので目障り
- **頻繁なアップデート**
- **セキュリティの脆弱性**
- 単純にパフォーマンスが悪い
- マスターしても単価が低いので仕事としての展望がない
- サーバを管理する必要がある
- VPS の料金が高い

要するに面倒くさがりなのと、出来るだけシンプルな環境を求めていたということになる。
WordPress は成熟しすぎた。「[UNIX という考え方](https://www.amazon.co.jp/dp/4274064069)」という本があるけど、その中で言う「人間による第二のシステム」の末期に差し掛かっている。

## 技術スタック

![ブログの執筆環境](02.jpg)

前回の反省を生かして以下の項目を満たす技術選定をした。

- **記事執筆**
  - オフライン対応
  - お気に入りの IDE（Intellij IDEA）使用可能性
  - マークダウン対応
  - 再利用性の高さ
  - プレビュー（ホットリロード）対応
- **技術**
  - サーバレス
  - SSG（Static Site Generator）
  - 無料
  - 仕事に活かせるか（モダンな技術）
  - デプロイの容易さ

今回選んだ技術に関してまだ完全に理解できていない部分も多いが、何度落ちても問題ない個人ブログであるから積極的に最新技術を試すサンドボックスとして活用していきたい。

### ドメイン -> Njalla

ドメインは匿名性を重視してスウェーデン発のサービスである [Njalla](https://njal.la) を使ってみた。

通常であればドメインを取得する際に連絡先住所を登録してから Whois で照会後にレジストラが代理で連絡先を公開してくれる。
しかし、これでもレジストラに個人情報が渡ってしまうと言う意味では匿名性は担保されていない。

Njalla の場合はドメインの所有権が Njalla 自身にあり、サービス利用者はそれを借りる形で完全なる匿名性を実現している。
利用者は一切個人情報を登録する必要がない（メールアドレスの代わりに XMPP、クレジットカードの代わりに仮想通貨を使用できる）。

当ブログでは匿名性を大きなテーマとして扱っていきたいので試しに使ってみた。
各サービス間で VPN サーバを切り替えたり、支払い方法などを変えたり匿名性を保つための工夫もしたが長くなるので割愛する。

### フロントエンド -> Gatsby.js & TypeScript

マークダウン対応や SSG を実現するためにフロントエンドのフレームワークは [Gatsby.js](https://github.com/gatsbyjs/gatsby) を選択した。
主要なフレームワークとして [Next.js](https://github.com/vercel/next.js) もあるが SSR は使わないのと強力なプラグイン機構が使えるという面で Gatsby.js の方がブログ向きかなと判断した。

また、せっかくなので JavaScript ではなく TypeScript で書いていく（勝手が分からないけど）。
今のところ GraphQL から自動的に型定義を吐き出す設定にしているが各コンポーネントで定義していないので順次対応させていきたい。

### ホスティング -> Netlify

フロントエンドのデプロイ先としては [Netlify](https://www.netlify.com) を選択した。
GitHub と連携させることにより、わずか 5 分足らずで全世界にブログを公開できた。

このブログの規模であれば無料枠でやっていけるので経済的にも助かっている。結局、ブログの運営費はドメイン代の年間$15 だけである。
WordPress 時代は ConoHa VPS に年間 1.5 万円ほど納めていたから節約になった。

他にも Vercel や Firebase Hosting など似たようなサービスがあるが、どれも無料で試せるのでスピードの比較をしてみたい。
Netlify は日本に CDN のエッジを持っていないのはちょっと気になるポイント。今のところ不便は感じていないが。

### その他のこだわりポイント

#### Twemoji

ブログカードとアイキャッチには絵文字を使用している。これは [Twemoji](https://github.com/twitter/twemoji) を使って絵文字を SVG に変換したものだ。
このアイディアは Zenn から拝借した。
記事の Frontmatter に絵文字を設定するだけで面倒なアイキャッチ画像を作る手間が省けるので時間の節約に繋がった。

#### styled-components

React のスタイリング方法に関しては様々な論争の元になる。当ブログでは CSS in JS の雄である [styled-components](https://github.com/styled-components/styled-components) を使用している。
今後、Atomic design と関数型の概念を取り入れていきたいのでもっとも相性が良い CSS-in-JS を選択した。

ただし、レンダリングコストの問題もあり、styled-components より後発の [emotion](https://github.com/emotion-js/emotion) か [linaria](https://github.com/callstack/linaria) の導入を検討している。

#### カテゴリのサブモジュール化

当ブログの記事は各カテゴリごとにサブモジュールを作成している。

- [fashion](https://github.com/ktnkk/blog.fashion)
- [life](https://github.com/ktnkk/blog.life)
- [onsen](https://github.com/ktnkk/blog.onsen)
- [tech](https://github.com/ktnkk/blog.tech)

サブモジュール化することで再利用性を高められる。
例えば後発で Gatsby.js に変わる新たなフレームワークが出てきたら外枠さえできればサブモジュールと紐づけることですぐに記事の移植ができる。

他にもブログ本体と記事のプルリクを分けられるので運用もしやすい。

## 開発ロードマップ

完璧を目指すよりもまずは終わらせることが先決であるから未完成だけどリリースした。
時間が足りず、後回しにしている機能も含めて洗いざらい吐き出しておく。

### 必ず欲しい機能

- ~~画像フォーマットの WebP 対応~~（[#8](https://github.com/ktnkk/blog/issues/8)）
- 動的な OGP 画像の生成（[#10](https://github.com/ktnkk/blog/issues/10)）
- シェルスクリプトで対話的に Frontmatter を自動生成（[#11](https://github.com/ktnkk/blog/issues/11)）
- サイドバーにプロフィール（[#100](https://github.com/ktnkk/blog/issues/100)）
- 記事のサイドバーにスクロールスパイ付き目次（[#101](https://github.com/ktnkk/blog/issues/101)）
- タグ機能の追加（[#102](https://github.com/ktnkk/blog/issues/102)）
- [Algolia](https://www.algolia.com/) を使った全文検索（[#103](https://github.com/ktnkk/blog/issues/103)）
- ~~[textlint](https://github.com/textlint/textlint) で記事校正の自動化~~（[#104](https://github.com/ktnkk/blog/issues/104)） -> [記事](/3445436)
- [Husky](https://github.com/typicode/husky) でコミット前検証（[#105](https://github.com/ktnkk/blog/issues/105)）
- Atomic design の導入（[#106](https://github.com/ktnkk/blog/issues/106)）
- [Storybook](https://github.com/storybookjs/storybook) の導入（[#107](https://github.com/ktnkk/blog/issues/107)）
- Netlify へのデプロイ時にキャッシュを使用（[#108](https://github.com/ktnkk/blog/issues/108)）
- [Jest](https://github.com/facebook/jest) を使ったテスト（[#109](https://github.com/ktnkk/blog/issues/109)）
- ~~IntelliJ IDEA で Docker 内の node を参照できない問題の解決~~（[#110](https://github.com/ktnkk/blog/issues/110)） -> Vim に乗り換えるのでクローズ
- ~~下書き機能の追加~~（[#116](https://github.com/ktnkk/blog/issues/116)）-> [記事](/7976845)
- ~~ページネーションの追加~~（[#159](https://github.com/ktnkk/blog/issues/159)）-> [記事](/8443869)

### 検討中の機能

- ~~md の mdx 化~~（[#111](https://github.com/ktnkk/blog/issues/111)） -> やっぱり md で書く
- styled-components を Emotion に置き換える（[#112](https://github.com/ktnkk/blog/issues/112)）
- GitHub の Issue または Discussion を使ったコメント機能の追加（[#113](https://github.com/ktnkk/blog/issues/113)）
- GitHub Actions と GitHub Packages で Docker image の配布（[#114](https://github.com/ktnkk/blog/issues/114)）
- 記事の履歴を参照できるようにする（[#115](https://github.com/ktnkk/blog/issues/115)）

[Roadmap to 2.0.0](https://github.com/ktnkk/blog/projects/2)

## さいごに

自前の IDE で IdeaVim を使って執筆できるようになったので、かなり DX は上昇した。むしろ記事を書くだけなら Vim でも良いかな。

今回実装した機能とこれから実装する機能は実際の業務でも使えるものなので一石二鳥という感じがする。
業務で習得した技術もこちらに応用させやすいから良い循環になりそう。

そもそもなぜブログを書くのかについてはまた別の記事に纏める予定。
