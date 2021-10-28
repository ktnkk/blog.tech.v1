---
title: "Gatsbyにページネーションを実装する"
date: "2021-10-28T21:48:56.421Z"
category: "t"
description: ""
emoji: "📒"
slug: "8443869"
published: true
---

## モチベーション

* 記事数が多くなったため1ページあたりのデータ量を減らしたい
* スクロールする手間を減らしたい
* UXを向上させたい

ページネーションではなくAjaxを使った無限スクロールもあるけど、個人的にあまり好きではないので選択肢から除外した。
Instagramみたいな魅せる画像系とは相性が良いと思うが、ブログみたいにピンポイントで記事を探しているときには融通が効かない（最初の記事をみたいのに無駄なスクロールをする必要がある）のであまり向いてないと思う。

また、Gatsbyにはページネーションを簡単に実装できる下記のプラグインがある。

* [gatsby-awesome-pagination](https://github.com/GatsbyCentral/gatsby-awesome-pagination)
* [gatsby-paginate](https://github.com/pixelstew/gatsby-paginate)
* [gatsby-plugin-paginate](https://github.com/kbariotis/gatsby-plugin-paginate)

今回はプラグインを使用しないで実装しようと思う。
なぜなら上記のプラグインは公式ではないからサポートが途切れる可能性があるし、ページネーション自体は大した実装ではないからだ。
それにややこしい`gatsby-node`関連の知見を溜めたいという気持ちもある。

[[i | 参考]]
| [Gatsby.js公式ドキュメント](https://www.gatsbyjs.com/docs/adding-pagination/)

## 開発環境

```shell:title=Bash {outputLines: 2-4, 6}{}
gatsby -v
Gatsby CLI version: 3.14.0
Gatsby version: 3.14.3

yarn info react version
17.0.2
```

## 実装

### パスの修正

ページネーションを使うということはページが複数あるということでそれらは動的に生成されなければならない。
そのため、静的ページを生成するフォルダ（`src/pages/*`）から動的ページを生成する際のテンプレート置き場のフォルダ（`src/templates/*`）に対象ファイルを移動させる必要がある。

当ブログはトップページで記事一覧を表示しているが`src/pages/index.tsx`で管理していた。
これをまずは`src/templates/index.tsx`にリネームする。

静的ページでも強引にページネーションを実装することは可能だが**記事数が多くなったため1ページあたりのデータ量を減らしたい**というメリットを享受できないのでオススメしない。

今回は全記事一覧とカテゴリ別記事一覧ページにページネーションを実装していく。

* 全記事一覧ページ -> `src/templates/index.tsx`
* カテゴリ別記事一覧ページ -> `src/templates/categories.tsx`

***

### ページの生成

次に`gatsby-node.js`でページを動的に生成する。

#### 全記事一覧ページの場合

```javascript:title=gatsby-node.js
exports.createPages = async ({ graphql, actions, reporter }) => {
  // ...
  
  const posts = result.data.allMarkdownRemark.edges;
  const postsPerPage = 8;
  const numPages = Math.ceil(posts.length / postsPerPage);

  Array.from({ length: numPages }).forEach((_, i) => {
    createPage({
      path: i === 0 ? `/` : `/${i + 1}`,
      component: path.resolve("./src/templates/index.tsx"),
      context: {
        limit: postsPerPage,
        skip: i * postsPerPage,
        numPages,
        currentPage: i + 1,
        hasPrevPage: i !== 0,
        hasNextPage: i !== numPages - 1
      }
    });
  });
}
```

1. 1ページあたりの記事数（8件）を設定（`postPerPage`）
2. 全記事数から1ページあたりの記事数を割ることでページ数を算出（`numPages`）
3. ページ数からArrayインスタンスを作成し、`forEach`で回す

全記事を取得して分割するだけなので特に難しいことはないが、いくつかポイントがある。

* `path`で三項演算子を利用して1ページしかない場合のパスを定義する
* `context`で後からコンポーネントで使用するプロパティを定義する
  * `context`で定義した値はpageContextで呼び出したり、Graphqlの変数（`$limit, $skip`）として使える

#### カテゴリ別記事一覧ページの場合

```javascript:title=gatsby-node.js
exports.createPages = async ({ graphql, actions, reporter }) => {
  // ...

  const posts = result.data.allMarkdownRemark.edges;

  let categories = [];
  posts.forEach(post => {
    if (post.node.frontmatter.category) {
      categories.push(post.node.frontmatter.category);
    }
  });

  categories = new Set(categories);
  const postsPerPage = 8;

  categories.forEach(category => {
    let categoryPosts = posts.filter(post => {
      return post.node.frontmatter.category === category;
    });
    const catNumPages = Math.ceil(categoryPosts.length / postsPerPage);
    Array.from({ length: catNumPages }).forEach((_, i) => {
      createPage({
        path: i === 0 ? `/${category}` : `/${category}/${i + 1}`,
        component: path.resolve("src/templates/categories.tsx"),
        context: {
          category,
          limit: postsPerPage,
          skip: i * postsPerPage,
          numPages: catNumPages,
          currentPage: i + 1,
          hasPrevPage: i !== 0,
          hasNextPage: i !== catNumPages - 1
        }
      });
    });
  });
}
```

全記事一覧ページの場合と被っている部分もあるが、違う点はカテゴリ毎にページを生成するという点だ。
カテゴリの配列を作ってからそれを`forEach`で回す。更にその中でカテゴリ毎の記事数を取得して前述したようにページを作成する。

***

### 必要な記事だけを取得

これで下準備ができた。この時点ではページ自体は生成されているが中身は変わらないので`templates/*`でGraphQLクエリに値を渡し、必要な記事数だけを取得できるようにする。

#### 全記事一覧ページの場合

```diff:title=index.tsx
// ...

export const pageQuery = graphql`
- query Index {
+ query Index($skip: Int!, $limit: Int!) {
    site {...}
-   allMarkdownRemark(sort: { fields: [frontmatter___date], order: DESC }) {
+   allMarkdownRemark(sort: { fields: [frontmatter___date], order: DESC }, limit: $limit, skip: $skip) {
      edges {...}
    }
  }
`;
```

`query Index`の仮引数に`gatsby-node.js`で定義した変数を入れる。
それを`allMarkdownRemark`で実引数として使うことで動的に必要なページのみを生成することができる。

この辺の挙動は実際に`http://localhost:8000/__graphiql`にアクセスして確かめると分かりやすい。

例：記事の絵文字を日付順に3件目から（2件スキップして）2件取得

```graphql:title=GraphQL
query MyQuery {
  allMarkdownRemark(
    limit: 2
    skip: 2
    sort: {fields: frontmatter___date, order: DESC}
  ) {
    edges {
      node {
        frontmatter {
          emoji
        }
      }
    }
  }
}
```

```json:title=JSON
{
  "data": {
    "allMarkdownRemark": {
      "edges": [
        {
          "node": {
            "frontmatter": {
              "emoji": "💡"
            }
          }
        },
        {
          "node": {
            "frontmatter": {
              "emoji": "📝"
            }
          }
        }
      ]
    }
  },
  "extensions": {}
}
```

デフォルトで様々なクエリを使えるのがGatsbyの大きな利点の1つであるから積極的に試してみたいところだ（まだまだ使いこなせていない）。

#### カテゴリ別記事一覧ページの場合

```diff:title=categories.tsx
export const pageQuery = graphql`
- query CategoryTemplate($category: String) {
+ query CategoryTemplate($category: String, $skip: Int!, $limit: Int!) {
    site {...}
    allMarkdownRemark(
      sort: { fields: [frontmatter___date], order: DESC }
      filter: { frontmatter: { category: { eq: $category } } }
+     limit: $limit
+     skip: $skip
    ) {
      edges {...}
    }
  }
`;
```

全記事一覧ページとほとんど同じだが、カテゴリ毎に記事を取得するために`filter`が設けられている。

この段階でURLを入力するとちゃんとページ分けされていることが確認できる。
次にページネーションのUIコンポーネントを作成していく。

***

### UIコンポーネントの作成

コンポーネントでは`gatsby-node.js`で定義した`context`を利用する。
ページネーションのデザインは好みが別れるので参考程度に考えていただきたい。

当ブログはスタイリングに`styled-components`を使用しているが、下記はそれ以外の部分。

```typescript:title=pagination.tsx
import * as React from "react";
import { Link } from "gatsby";
import styled from "styled-components";

const Pagination = ({ numPages, currentPage, hasNextPage, hasPrevPage, pagePath, className }) => {

  // 前のページと次のページがなければ表示しない
  if (!hasNextPage && !hasPrevPage) {
    return null;
  }

  let pageNumbers = [];

  // 総ページ数と開いているページ数に応じて表記を変える
  if (numPages <= 7) {
    for (let i = 1; i <= numPages; i++) {
      pageNumbers.push(i);
    }
  } else if (currentPage <= 4) {
    pageNumbers = [1, 2, 3, 4, 5, "…", numPages];
  } else if (currentPage < numPages - 3) {
    pageNumbers = [1, "…", currentPage - 1, currentPage, currentPage + 1, "…", numPages];
  } else {
    pageNumbers = [1, "…", numPages - 4, numPages - 3, numPages - 2, numPages - 1, numPages];
  }

  return (
    <div className={className}>
      <div className="pagination__prev">
        <Link
          rel="prev"
          to={hasPrevPage ? pagePath(currentPage - 1) : "/#"}
          // 前ページが存在しなければ`-disable`セレクタを追加
          className={`pagination__prev-link ${!hasPrevPage && "-disable"}`}
        >
          &lt; Prev
        </Link>
      </div>
      <ul className="pagination__list-container">
        {pageNumbers.map((pn, i) => (
          <li key={`pageNum-${i}`}>
            {pn === "…" ? (
              <span>…</span>
            ) : (
              // カレントページの場合は`selected`セレクタを追加
              <Link to={pagePath(pn)} className={pn === currentPage ? "selected" : ""}>
                {pn}
              </Link>
            )}
          </li>
        ))}
      </ul>
      <div className="pagination__next">
        <Link
          rel="next"
          to={hasNextPage ? pagePath(currentPage + 1) : "/#"}
          // 次ページが存在しなければ`-disable`セレクタを追加
          className={`pagination__next-link ${!hasNextPage && "-disable"}`}
        >
          Next &gt;
        </Link>
      </div>
    </div>
  );
};

// ...
```

まずデストラクチャリングで`pageContext`からページネーションに必要な要素を取り出す。
このコンポーネントは現時点ではどこにも使っていないけれど後から使う想定で書いていく。

1つずつ説明するのも冗長なのでポイントとなる箇所にコメントを記載した。
総ページ数と開いているページ数に応じて表記を変える部分の数値を変えると色々なバリエーションが生まれると思う。

あと、上記では`&lt; Prev`と`Next &gt;`で前後ボタンを常時表示しているが、`{hasPrevPage && "< Prev"}`みたいにページが存在する場合のみ表示させる方法もある。

次にスタイリング。

```typescript:title=pagination.tsx
// ...

const StyledPagination = styled(Pagination)`
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 16px 0;

  .pagination__prev {
    flex-grow: 1;
    text-align: left;
  }

  .pagination__next {
    flex-grow: 1;
    text-align: right;
  }

  .pagination__prev-link,
  .pagination__next-link {
    color: #539bf5;
    font-size: 1rem;
    vertical-align: middle;
    border: 1px solid transparent;
    padding: 4px 12px;
    border-radius: 6px;
    transition: border-color 0.2s cubic-bezier(0.3, 0, 0.5, 1);

    @media screen and (max-width: 567px) {
      display: none;
    }

    &:hover,
    &:focus {
      border-color: #444c56;
      transition-duration: 0.1s;
      outline: 0;
    }

    &.-disable {
      pointer-events: none;
      color: #545d68;
    }
  }

  .pagination__list-container {
    margin: 0;

    li {
      display: inline-block;
      margin: 5px;

      a,
      span {
        min-width: 32px;
        font-size: 1rem;
        color: #adbac7;
        vertical-align: middle;
        font-weight: normal;
        line-height: 1rem;
        display: inline;
        padding: 4px 10px;
        border: 1px solid transparent;
        border-radius: 6px;
        transition: border-color 0.2s cubic-bezier(0.3, 0, 0.5, 1);

        &:hover,
        &:focus {
          border-color: #444c56;
          transition-duration: 0.1s;
        }
      }

      a.selected {
        color: #cdd9e5;
        background-color: #316dca;
        border-color: transparent;
        pointer-events: none;
      }

      span {
        pointer-events: none;

        &:hover,
        &:focus {
          border-color: transparent;
        }
      }
    }
  }
`;

export default StyledPagination;
```

CSS-in-JSかCSS Moduleかによる記法の違いはあれど共通するポイントとしては`-disable`と`selected`、`span`（…）の各セレクタでは`pointer-events: none;`で押せなくする必要がある。
それ以外は自分の好みに合わせて書き換えれば良いかも。

***

### コンポーネントを使う

上記で作成したコンポーネントを実際に使ってみる。

#### 全記事一覧ページの場合

```diff:title=index.tsx
// ...

const Index = ({ data, pageContext, location }) => {
  // ...

+ const { currentPage, hasNextPage, hasPrevPage, numPages } = pageContext;
+ const postPagePath = page => (page <= 1 ? `/` : `/${page}/`);

  return (
    <Layout location={location} title={siteTitle}>
      <SEO title="" />
      <Helmet>
        <link rel="canonical" href="https://ktnkk.com" />
      </Helmet>
      <HomeJsonLd />
      <CategoryMenu location={location} currentPage={currentPage} />
      <PostsContainer>
        {posts.map(({ node }) => {
          return <PostCard key={node.fields.slug} node={node} />;
        })}
      </PostsContainer>
+     <Pagination
+       pagePath={postPagePath}
+       numPages={numPages}
+       currentPage={currentPage}
+       hasNextPage={hasNextPage}
+       hasPrevPage={hasPrevPage}
+     />
    </Layout>
  );
};
```

`pageContext`からデストラクチャリングで各プロパティを取り出す。
そして、まだ定義していなかった関数`postPagePath`を作成。

TSX内ではコンポーネントタグに必要なプロパティを持たせる。これで完成。

#### カテゴリ別記事一覧ページの場合

これもほぼ同じ書き方だがパスが違うのだけ修正する。

```typescript
const postPagePath = page => (page <= 1 ? `/${categorySlug}` : `/${categorySlug}/${page}/`);
```

***

### おまけ

当ブログではカテゴリを円形のボタンで切り替えられるが、ページを変えるとactiveでなくなる現象が発生した。

```typescript:title=categoryMenu.tsx
const pathInclude =
  currentPage === 1
    ? path === catLink
    : catLink === "/"
    ? path === `/${currentPage}/`
    : path === `${catLink}/${currentPage}/`;
```

このように書くことで解決した。もし何かactiveにすることでスタイルを変えている場合は対応する必要があるかもしれない。
もしくは現時点ではちょっと理解が乏しいけどHooksを使うことで対応できるかも。良い解決策が思いついたら追記する。

## さいごに

`gatsby-node`を使ったページの生成に苦労したがそれを理解することで今後様々な動的ページの表現方法が広がるのでやってよかったと思う。
けっこう癖があるけど使いこなせれば便利なツールだということが分かった。

[[i | 関連リンク]]
| * [Issue](https://github.com/ktnkk/blog/issues/159)
| * [Pull request](https://github.com/ktnkk/blog/pull/161)
