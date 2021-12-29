---
title: "ktnkk.log のマークダウン取扱説明書"
date: "2021-10-13T23:36:44.284Z"
category: "t"
description: ""
emoji: "Ⓜ️"
slug: "7967644"
published: true
---

<!-- markdownlint-disable -->

当ブログのマークダウン記法とそれに対応する表示をテストするために全て書き留めておく。機能追加していく中でデグレ起こす可能性があるので。

実際はあまり使ってない記法もあるけど…。これを機に活用していきたい。

---

## Heading

```markdown:title=Markdown
## Heading level 2
### Heading level 3
#### Heading level 4
##### Heading level 5
```

## Heading level 2

### Heading level 3

#### Heading level 4

##### Heading level 5

---

## List

### ul

```markdown:title=Markdown
- first item
  - (first item)
- second item
  - (second item)
- third item
  - (third item)
```

- first item
  - (first item)
- second item
  - (second item)
- third item
  - (third item)

### ol

```markdown:title=Markdown
1. first item
2. second item
3. third item
4. fourth item
5. fifth item
```

1. first item
2. second item
3. third item
4. fourth item
5. fifth item

---

## Inline text

```markdown:title=Markdown
*stress emphasis*
**strong importance, seriousness, or urgency**
~~text that has been deleted from a document~~
`short fragment of computer code`
[This is my homepage](https://ktnkk.com/)
<https://ktnkk.com/>
```

_stress emphasis_

**strong importance, seriousness, or urgency**

~~text that has been deleted from a document~~

`short fragment of computer code`

[This is my homepage](https://ktnkk.com/)

<https://ktnkk.com/>

---

## Blockquotes

```markdown:title=Markdown
> Work as intensely as you play and play as intensely as you work.
> Also, don't be content with a narrow range of skills.
> A hacker who's a system administrator, on the other hand, is likely to be quite skilled at script programming and web design.
> Hackers don't do things by halves; if they invest in a skill at all, they tend to get very good at it.
```

> Work as intensely as you play and play as intensely as you work.
> Also, don't be content with a narrow range of skills.
> A hacker who's a system administrator, on the other hand, is likely to be quite skilled at script programming and web design.
> Hackers don't do things by halves; if they invest in a skill at all, they tend to get very good at it.

---

## Table

```markdown:title=Markdown
| Countries | Capitals         | Population  | Language |
| --------- | ---------------- | ----------- | -------- |
| USA       | Washington, D.C. | 309 million | English  |
| Sweden    | Stockholm        | 9 million   | Swedish  |
```

| Countries | Capitals         | Population  | Language |
| --------- | ---------------- | ----------- | -------- |
| USA       | Washington, D.C. | 309 million | English  |
| Sweden    | Stockholm        | 9 million   | Swedish  |

---

## Image

```markdown:title=Markdown
![🐿](01.jpg)
```

![🐿](01.jpg)

---

## Horizontal Rule

```markdown:title=Markdown
***
```

---

## Code

### Basic

<div class="gatsby-code-title">Markdown</div>
<div class="gatsby-highlight" data-language="markdown"><pre class="language-text"><code class="language-text">```javascript:title="gatsby-config.js
plugins: [
  {
    resolve: `gatsby-remark-prismjs`,
    options: {
      classPrefix: "language-",
      inlineCodeMarker: null,
      aliases: {},
      showLineNumbers: false,
      noInlineHighlight: false,
      prompt: {
        user: "",
        host: "",
        global: true,
      },
      escapeEntities: {}
    }
  }
]
```</code></pre></div>

```javascript:title=gatsby-config.js
plugins: [
  {
    resolve: `gatsby-remark-prismjs`,
    options: {
      classPrefix: "language-",
      inlineCodeMarker: null,
      aliases: {},
      showLineNumbers: false,
      noInlineHighlight: false,
      prompt: {
        user: "",
        host: "",
        global: true,
      },
      escapeEntities: {}
    }
  }
]
```

### Line highlighting

<div class="gatsby-code-title">Markdown</div>
<div class="gatsby-highlight" data-language="markdown"><pre class="language-text"><code class="language-text">```java{4,6-7}:title=Console.java
public class Console {
    public static void main(String[] args) {
        System.out.println("3");
        System.out.println("4");
        System.out.println("5");
        System.out.println("6");
        System.out.println("7");
        System.out.println("8");
    }
}
```</code></pre></div>

```java{4,6-7}:title=Console.java
public class Console {
    public static void main(String[] args) {
        System.out.println("3");
        System.out.println("4");
        System.out.println("5");
        System.out.println("6");
        System.out.println("7");
        System.out.println("8");
    }
}
```

### Shell prompt

<div class="gatsby-code-title">Markdown</div>
<div class="gatsby-highlight" data-language="markdown"><pre class="language-text">
<code class="language-text">```shell:title=Bash {outputLines: 2, 4}{}
echo "shell prompt"
shell prompt
echo $USER
ktnkk
```</code></pre></div>

```shell:title=Bash {outputLines: 2, 4}{}
echo "shell prompt"
shell prompt
echo $USER
ktnkk
```

<div class="gatsby-code-title">Markdown</div>
<div class="gatsby-highlight" data-language="markdown"><pre class="language-text"><code class="language-text">```shell:title=Bash {outputLines: 2}{promptUser: root}
echo $USER
root
```</code></pre></div>

```shell:title=Bash {outputLines: 2}{promptUser: root}
echo $USER
root
```

### Diff code blocks

<div class="gatsby-code-title">Markdown</div>
<div class="gatsby-highlight" data-language="markdown"><pre class="language-text"><code class="language-text">```diff:title=Dockerfile
- FROM node:16.11.0-bullseye-slim
+ FROM node:16.11.1-bullseye-slim
  LABEL maintainer="ktnkk@pm.me"
  LABEL version="1.0.0"
```</code></pre></div>

```diff:title=Dockerfile
- FROM node:16.11.0-bullseye-slim
+ FROM node:16.11.1-bullseye-slim
  LABEL maintainer="ktnkk@pm.me"
  LABEL version="1.0.0"
```

---

## KaTeX

### Centered

```markdown:title=Markdown
$$
i \hbar \dfrac{\partial}{\partial t} \psi(r,t)
= \left(- \dfrac{\hbar^2}{2m} \nabla^2+ V(r,t) \right) \psi(r,t)
$$
```

<!-- textlint-disable ja-technical-writing/sentence-length -->

$$
i \hbar \dfrac{\partial}{\partial t} \psi(r,t)
= \left(- \dfrac{\hbar^2}{2m} \nabla^2+ V(r,t) \right) \psi(r,t)
$$

<!-- textlint-enable -->

### Left-aligned

```markdown:title=Markdown
$
\rho \left\{\dfrac{\partial{\boldsymbol{v}}}{\partial{t}} + \left(\boldsymbol{v} \cdot \nabla \right) \boldsymbol{v}\right\}
= -\nabla p + \mu \nabla^2 \boldsymbol{v} + \rho \boldsymbol{f}
$
```

<!-- textlint-disable ja-technical-writing/sentence-length -->

$
\rho \left\{\dfrac{\partial{\boldsymbol{v}}}{\partial{t}} + \left(\boldsymbol{v} \cdot \nabla \right) \boldsymbol{v}\right\}
= -\nabla p + \mu \nabla^2 \boldsymbol{v} + \rho \boldsymbol{f}
$

<!-- textlint-enable -->

### Inline

```markdown:title=Markdown
Let $\begin{pmatrix} 3 \\ 4 \\ 5 \end{pmatrix}$ be the matrix in the text.
```

Let $\begin{pmatrix} 3 \\ 4 \\ 5 \end{pmatrix}$ be the matrix in the text.

---

## Graphviz

```text:title=Markdown
graph ER {
  graph [
    charset = "UTF-8";
    bgcolor = "#343434",
    fontcolor = lightgrey,
    margin = 0.2,
  ];
  layout=neato
  node [
    shape=box,
    fontcolor=white
  ];
  course; institute; student;
  node [
    shape=ellipse
  ];
  {node [label="name"] name0; name1; name2;}
  code; grade; number;
  node [
    shape=diamond,
    style=filled,
    color=darkgrey
  ];
  "C-I"; "S-C"; "S-I";

  name0 -- course;
  code -- course;
  course -- "C-I" [color=lightgrey];
  "C-I" -- institute [color=lightgrey];
  institute -- name1;
  institute -- "S-I" [color=lightgrey];
  "S-I" -- student [color=lightgrey];
  student -- grade;
  student -- name2;
  student -- number;
  student -- "S-C" [color=lightgrey];
  "S-C" -- course [color=lightgrey];
}
```

```dot
graph ER {
  graph [
    charset = "UTF-8";
    bgcolor = "#343434",
    fontcolor = lightgrey,
    margin = 0.2,
  ];
  layout=neato
  node [
    shape=box,
    fontcolor=white
  ];
  course; institute; student;
  node [
    shape=ellipse
  ];
  {node [label="name"] name0; name1; name2;}
  code; grade; number;
  node [
    shape=diamond,
    style=filled,
    color=darkgrey
  ];
  "C-I"; "S-C"; "S-I";

  name0 -- course;
  code -- course;
  course -- "C-I" [color=lightgrey];
  "C-I" -- institute [color=lightgrey];
  institute -- name1;
  institute -- "S-I" [color=lightgrey];
  "S-I" -- student [color=lightgrey];
  student -- grade;
  student -- name2;
  student -- number;
  student -- "S-C" [color=lightgrey];
  "S-C" -- course [color=lightgrey];
}
```

※ 言語は`dot`を指定（レンダリングされてしまうので上記で書けず）。

---

## あとがき

基本的なマークダウンの記法通りで特に真新しさはない。
違うところはカスタムブロックとコードブロック、KaTaX、Graphviz くらいかな。

マークダウンの良さは互換性の高さだと思っている。
母艦が変わっても表示の一貫性が保たれているので、記録として残したい構造的な文章を書くのに打って付けだ。

だが、そう考えるとマークダウンの機能を拡張していくことはそのメリットを打ち消す気がする。
[MDX](https://mdxjs.com/)を使うことでマークダウン内にコンポーネントを書けるようにする方法もあるが、それはライブラリが React であるから成り立つことでもある。

今後は闇雲に機能を追加するのではなく、本当に必要な記法だけを厳選して追加したい。MDX を使うなら別のモジュールとして管理すればアリかな。

あと、[Graphviz](https://graphviz.org/)面白いね。これ（DOT 言語）をマスターすれば表を書くのにわざわざアプリを使わずに済むし、修正も容易だ。

## 20211229 追記

- [gatsby-remark-embed-youtube](https://www.gatsbyjs.com/plugins/gatsby-remark-embed-youtube/)
- [gatsby-plugin-twitter](https://www.gatsbyjs.org/packages/gatsby-plugin-twitter)
- [gatsby-remark-custom-blocks](https://www.gatsbyjs.com/plugins/gatsby-remark-custom-blocks/)

上記のプラグインを使った記法は廃止した。
いずれも UI をリッチに描画する機能だがマークダウンが持つ汎用性を殺してしまう。
それに textlint や markdownlint で引っかかるのが面倒だったのもある（そこだけ回避させるのは何か違う）。
結局、そういった小道具に頼るのではなく分かりやすい文章を書いていこう。

<!-- markdownlint-enable -->
