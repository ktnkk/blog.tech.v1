---
title: "Gatsby に下書き機能を実装する"
date: "2021-10-29T18:08:56.421Z"
category: "t"
description: ""
emoji: "📝"
slug: "7976845"
published: true
---

## モチベーション

現時点では`main`ブランチにプッシュすることでビルドが走り、記事が公開される設定になっている。
この機能はとても便利であるが、記事の下書きを保存するためにブランチを分ける必要があって面倒だった。

今回の機能追加で下書きフラグを設定してまだ書いている途中でもこまめにコミットできるようにしたい。
また、開発環境では下書きのプレビューを見られるように自動で判別できたら便利。

## 開発環境

```shell:title=Bash {outputLines: 2-4, 6}{}
gatsby -v
Gatsby CLI version: 3.14.0
Gatsby version: 3.14.3

yarn info react version
17.0.2
```

## 実装

### Frontmatter

まず、前提としてマークダウンファイルの Frontmatter に真偽値型で公開の是非を記載しておく。

```markdown{8}:title=index.md
---
title: "Gatsbyに下書き機能を実装する"
date: "2021-10-29T18:08:56.421Z"
category: "t"
description: ""
emoji: "📝"
slug: "7976845"
published: true
---
```

---

### ページ生成を制御

Gatsby のページ生成を司る`gatsby-node.js`にちょっと手を加えていく。
下書きを表示させないということはページ自体ももちろんだが、記事一覧ページの表示も制限させないといけない点に注意が必要である。
まずはページ自体のフィルタリング方法から。

```diff:title=gatsby-node.js
exports.createPages = async ({ graphql, actions, reporter }) => {
  const {createPage} = actions;
+ const isPublished = process.env.NODE_ENV === "production" ? [true] : [true, false];

  const result = await graphql(`
    {
      allMarkdownRemark(
        sort: { fields: [frontmatter___date], order: DESC }
        limit: 1000
+       filter: { frontmatter: { published: { in: [${isPublished}] } } }
      ) {
        edges {...}
      }
    }
  `);

  // ...
}
```

変数`published`は開発環境で下書きを表示するが、本番環境では表示しないという真偽値を求めている。
`process.env.NODE_ENV`は特に設定しなくても良い。
コマンド`gatsby develop`では`development`、`gatsby serve`では`production`が設定されるのでそれぞれ挙動を確認できる。

次に記事一覧ページのフィルタリング方法。

```diff:title=gatsby-node.js
exports.createPages = async ({ graphql, actions, reporter }) => {
  // ...

  Array.from({length: numPages}).forEach((_, i) => {
    createPage({
      path: i === 0 ? `/` : `/${i + 1}`,
      component: path.resolve("./src/templates/index.tsx"),
      context: {
+       isPublished,
        limit: postsPerPage,
        skip: i * postsPerPage,
        numPages,
        currentPage: i + 1,
        hasPrevPage: i !== 0,
        hasNextPage: i !== numPages - 1
      }
    });
  });

  // ...
}
```

先ほど定義した変数`published`を`context`に渡してあげる。
これを記事一覧ページのテンプレートで使用する。

```diff:title=index.tsx
export const pageQuery = graphql`
- query Index($skip: Int!, $limit: Int!) {
+ query Index($skip: Int!, $limit: Int!, $isPublished: [Boolean]!) {
    site {
      siteMetadata {
        title
      }
    }
    allMarkdownRemark(
      sort: { fields: [frontmatter___date], order: DESC }
+     filter: { frontmatter: { published: { in: $isPublished } } }
      limit: $limit
      skip: $skip
    ) {
      edges {...}
    }
  }
`;
```

`context`で渡された変数をフィルターとして使う。
これで完了。

`filter`で使用している比較演算子`in`は`in array`の略である。
他にも様々な演算子でフィルタリングができる。詳しくは[こちら](https://www.gatsbyjs.com/docs/graphql-reference/#filter)を参照。

## さいごに

これでこまめにコミットできるから間違ってファイルを消すことに怯えなくて済む。
今のところ下書きの記事自体が少ないから迷うことはないが、これから増えることが予想される。
余裕があれば下書き一覧ページの作成やラベルの貼り付けなどして下書き記事の管理をできるようにしたい。

## 関連リンク

- [Issue](https://github.com/ktnkk/blog/issues/116)
- [Pull request](https://github.com/ktnkk/blog/pull/174)
