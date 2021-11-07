---
title: "React Hooksを基礎から理解する（前編）"
date: "2021-11-06T15:31:56.421Z"
category: "t"
description: ""
emoji: "🪝"
slug: "2957598"
published: true
---

## プロローグ

React Hooksという状態管理などをするための機能がある。
これを今まで何となく触ってきたのでちゃんと理解したい。
ブログ程度であればなくても困らなかったけど、大規模なアプリ開発には必須だし、ちょっと小洒落たサイトを作る際も必要不可欠だ。

ざっと見た感じだと上手く使えればパフォーマンスの向上も期待できる。
出来ればこのブログにも今後取り入れてチューニングしたい。

目標としては新規開発でHooksを使う前提の設計ができるようになることと、既存コードの改修や機能追加（このパターンが多い）でコードの意図を適切に理解できること。

もちろん、全てのHooksを習得すべきだが、全く見たことがないものや使う頻度の低いものよりも使う確率が高いものから習得したほうが効率は良い。
そこで、幾つかピックアップして最低限それらのHooksは使いこなせるようになりたい。そこがスタートラインだ。

[[i | 今回対象のHooks]]
| * `useState`
| * `useEffect`
| * `useContext`
| * `useCallback`
| * `useMemo`
| * `useRef`

[[i | 後回しにするHooks]]
| * `useReducer`
| * `useImperativeHandle`
| * `useLayoutEffect`
| * `useDebugValue`

[[n | 参考にしたもの]]
| * [React公式ドキュメント](https://ja.reactjs.org/docs/react-api.html#hooks)
| * [Reactハンズオンラーニング 第2版](https://www.amazon.co.jp/React%E3%83%8F%E3%83%B3%E3%82%BA%E3%82%AA%E3%83%B3%E3%83%A9%E3%83%BC%E3%83%8B%E3%83%B3%E3%82%B0-%E7%AC%AC2%E7%89%88-%E2%80%95Web%E3%82%A2%E3%83%97%E3%83%AA%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E9%96%8B%E7%99%BA%E3%81%AE%E3%83%99%E3%82%B9%E3%83%88%E3%83%97%E3%83%A9%E3%82%AF%E3%83%86%E3%82%A3%E3%82%B9-Alex-Banks/dp/4873119383)
| * [基礎から学ぶ React/React Hooks](https://www.amazon.co.jp/%E5%9F%BA%E7%A4%8E%E3%81%8B%E3%82%89%E5%AD%A6%E3%81%B6-React-Hooks-asakohattori/dp/486354359X)

## 環境構築

[CodePen](https://codepen.io/)や[CodeSandbox](https://codesandbox.io/)を使って試す方法もある。
だが、やっぱり自前のエディターを使いたいので3分で環境を作る。
いつも通りDockerを使用する。

```dockerfile:title=Dockerignore
FROM node:17.0.1-bullseye-slim
WORKDIR /home/node/app
RUN apt update -q \
    && apt upgrade -y \
    && apt install -y \
       build-essential
EXPOSE 3000
```

```yaml:title=docker-compose.yml
version: "3.9"
services:
  dev:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: sandbox
    ports:
      - "3000:3000"
    volumes:
      - .:/home/node/app
    environment:
      - NODE_ENV=development
    tty: true
    stdin_open: true
```

```ignore:title=.gitignore
node_modules
```

下記のコマンドでコンテナを作成し、中に入る。

```shell:title=Zsh {outputLines: 2}{}
docker compose up --build -d

docker exec -it sandbox /bin/bash
```

次に`create-react-app`でTypeScript対応のReactアプリを作成する。

```shell:title=Bash {outputLines: 2, 4, 6, 8}{}
npx create-react-app my-app --template typescript

mv my-app/* my-app/.[^\.]* . && rm -rf my-app

rm -rf yarn.lock node_modules

yarn

export NODE_OPTIONS=--openssl-legacy-provider; yarn start
```

今回はNode.jsのバージョンを17に上げた関係で幾つかエラーが出た。
後々修正されるだろうけどいったんは上記の対応で動くようになった（[参考](https://nodejs.org/en/blog/release/v17.0.0/)）。

### バージョン

```shell:title=Bash {outputLines: 2-3, 5-6, 8}{}
node -v
v17.0.1

yarn info react version
17.0.2

yarn info typescript version
4.4.4
```

## 実践

### useState

まず初めに触れていくのはHooksの中でもっとも使用頻度の高いであろう`useState`である。
`useState`はステートフルな値と、それを更新するための関数を返す。
そして、Reactは再レンダー間で`setState`関数の同一性が保たれ、変化しないことを保証する。

基本構文は下記。

```javascript:title=JavaScript
const [state, setState] = useState(initialState);
```

配列の先頭には`state`値が格納されているが2番目の要素には関数が格納されている。
この関数を使って、コンポーネント内から`state`値を変更できる。
データが変更されるとHookは自身がフックされたコンポーネントを新しいデータで再描画する能力を持つ。

```javascript:title=JavaScript
const [state, setState] = useState(() => {
  const initialState = someExpensiveComputation(props);
  return initialState;
});
```

`initialState`は初回レンダー時に使われる`state`値である。
後続のレンダー時にはその値は無視される。
もし、初期`state`が高負荷な計算をして求める値である場合は、代わりに関数を渡せる。
この関数は初回のレンダー時にのみ実行される（遅延初期化）。

実際にカウントを計測できるコンポーネントを作成してみる。
注釈として下記では全ての機能を1つに纏めているが最善の書き方ではない。
本来はコンポーネント指向で細かく分けるほうが良い（その匙加減が難しいのだけれど）。

また、話は逸れるがアプリケーション内の全てのコンポーネントが状態を持つというのも好ましくはない。
なぜなら、機能追加やデバッグが難しくなるからである。
できる限り、アプリケーションの状態は1箇所で管理したほうが効率的である。
つまり、子コンポーネントは同じプロパティ値で呼び出すと必ず同じ結果を返す**純粋関数**として設計すべきとも言える。
例として、最上位のルートディレクトリで全ての状態を管理して、子コンポーネントにプロパティ値として渡してあげるみたいな構成は後々良い結果をもたらすであろう。

```typescript{7-8}:title=App.tsx
import "./App.css";
import { FC, useState, ChangeEvent } from "react";

const INITIAL_COUNT: number = 10;
const INITIAL_NAME: string = "Keiten Kiki";
const App: FC = () => {
  const [count, setCount] = useState<number>(INITIAL_COUNT);
  const [name, setName] = useState<string>(INITIAL_NAME);
  const countIncrement = (): void => setCount(prevCount => prevCount + 1);
  const countDecrement = (): void => setCount(prevCount => prevCount - 1);
  const countReset = (): void => setCount(INITIAL_COUNT);
  const handleChangeName = (e: ChangeEvent<HTMLInputElement>): void => {
    setName(e.target.value);
  };
  return (
    <div className="App">
      <p>
        Current count: <b>{count}</b>
        <br />
        Initial value: <b>{INITIAL_COUNT}</b>
      </p>
      <button onClick={countIncrement}>increment</button>
      <button onClick={countDecrement}>decrement</button>
      <button onClick={countReset}>reset</button>
      <p>
        Hello, <b>{name}!</b>
        <br />
        Initial value: <b>{INITIAL_NAME}</b>
      </p>
      <input type="text" onChange={handleChangeName} />
    </div>
  );
};

export default App;
```

***

### useEffect

Reactは描画を絶えず繰り返している。
それゆえ、DOMの書き換え、データの購読、タイマー、ロギング、あるいはその他の副作用が実行されるタイミングは描画完了後でなければならない（!Reactのレンダーフェーズ）。
さもなくば、UIにまつわるややこしいバグや非整合性を引き起こしてしまう。

そのような場面で使うべきHooksは`useEffect`である。
`useEffect`へ渡された関数は描画結果の反映後に動作する。

> 副作用とは React の純粋に関数的な世界から命令型の世界への避難ハッチであると考えてください。

公式ドキュメントには副作用の説明は上記のように記されている。
しかし、抽象的で何とも分かりにくい。
簡単に言い換えると**副作用とは描画の一部ではない処理**のことである。
その視点で考えると`alert`や`console.log`などのAPIを呼び出す際には`useEffect`を使うと良い。
また、確実に描画が完了するのを待ってから実行したい処理を記述するためにも使用できる。

基本構文は下記。

```javascript:title=JavaScript
useEffect(didUpdate);
```

ただ、副作用はしばしば、コンポーネントが画面から消える場合にクリーンアップする必要があるようなリソース（例えば購読やタイマーIDなど）を作成する。
その際にはクリーンアップ関数として戻り値を記述する。

```javascript:title=JavaScript
useEffect(() => {
  const subscription = props.source.subscribe();
  return () => {
    subscription.unsubscribe();
  };
});
```

デフォルトの動作では、副作用関数はレンダーの完了時に毎回実行されてしまう。
これにより、コンポーネントの依存配列のうちのいずれかが変化した場合に毎回副作用が再作成される。
これは当然ながらパフォーマンス的に良くない。
それぞれの出力は別々のプロパティに連動すべきである。
これを実現するために**依存配列**を使用する。
依存配列を省略するとコンポーネントがレンダリングされるたびに副作用関数が実行されてしまうため基本的には省略しない。

```javascript:title=JavaScript
useEffect(
  () => {
    const subscription = props.source.subscribe();
    return () => {
      subscription.unsubscribe();
    };
  },
  [props.source],
);
```

`useEffect`の第2引数に副作用が実行されるトリガーとなるプロパティを指定する。
また、空の配列を依存配列に指定可能であり、その際はコンポーネントの初回描画時のみ副作用が実行される。

以下はカウントを計測するコンポーネントである。
UIの描画が終わるタイミングでタイトルを更新する。

```typescript{10}:title=App.tsx
import "./App.css";
import { FC, useState, useEffect } from "react";

const INITIAL_COUNT: number = 0;
const App: FC = () => {
  const [count, setCount] = useState<number>(INITIAL_COUNT);
  const callbackFunc = (): void => {
    document.title = `It's been clicked ${count} times.`;
  };
  useEffect(callbackFunc, [count]);
  const countIncrement = (): void => {
    setCount(prevCount => prevCount + 1);
  };
  const countReset = (): void => {
    setCount(INITIAL_COUNT);
  };
  return (
    <div className="App">
      <p>Current count: {count}</p>
      <button onClick={countIncrement}>+1</button>
      <button onClick={countReset}>reset</button>
    </div>
  );
};

export default App;
```

続いてクリーンアップ関数ありの`useEffect`である。
ボタンをクリックすると`setInterval`を使用してタイムカウントが始まる。
そして、再びボタンを押すとクリーンアップ関数が実行されてタイマーがクリアされる。

```typescript{22}:title=App.tsx
import "./App.css";
import { FC, useState, useEffect } from "react";

const INITIAL_COUNT: number = 0;
const Timer: FC = () => {
  const [count, setCount] = useState<number>(INITIAL_COUNT);
  const countReset = (): void => {
    setCount(INITIAL_COUNT);
  };
  const countIncrement = (): void => {
    setCount(prevCount => prevCount + 1);
    console.log("Count-up: +1");
  };
  const callbackFunc = (): (() => void) => {
    alert("Side effect function has been executed.");
    const timer = setInterval(countIncrement, 1000);
    return () => {
      console.log("The timer has been deleted.");
      clearInterval(timer);
    };
  };
  useEffect(callbackFunc, []);
  return (
    <div className="App">
      <p>Current count: {count}</p>
      <button onClick={countReset}>reset</button>
    </div>
  );
};

const App: FC = () => {
  const [display, toggleDisplay] = useState<boolean>(false);
  const handleToggleDisplay = (): void => {
    toggleDisplay(!display);
  };
  return (
    <>
      <button onClick={handleToggleDisplay}>{display ? "Hide the timer" : "Show the timer"}</button>
      {display && <Timer />}
    </>
  );
};

export default App;
```

***

### React.memo

デフォルトでは`props`オブジェクト内の複雑なオブジェクトは浅い比較のみが行われる。
もし、あるコンポーネントが同じ`props`を与えられたときに同じ結果を描画するなら、結果を記憶してくれたほうが良いに決まっている。
そのために使用するのがトップレベル関数である`memo`である。

そもそもメモ化とは情報工学の一般的な用語で、パフォーマンス改善のために計算結果をキャッシュすることを意味する。

`memo`を使うことではコンポーネントの再描画をスキップし、最後の描画結果を再利用できる（プロパティが更新されない限り再レンダリングはされない）。
レンダリングコストが高いコンポーネントや頻繁に再レンダリングされるコンポーネント内の子コンポーネントに使用すると効果的である。

ただし、`memo`関数でラップされるコンポーネントは純粋関数でなければならない点に注意が必要。
つまり、副作用がないコンポーネントのパフォーマンス改善に用いる。
また、コールバック関数を受け取ったコンポーネントは`memo`を利用しても必ず再描画されるが、後述する`useCallback`と併用することでスキップできる。

下記は具体的な使用例。

```typescript{9}:title=App.tsx
import "./App.css";
import { FC, memo, useState } from "react";

type CountResultProps = {
  text: string;
  countState: number;
};

const CountResult: FC<CountResultProps> = memo(({ text, countState }: { text: string; countState: number }) => {
  console.log(`The ${text} button has been clicked.`);
  return (
    <p>
      {text}: {countState}
    </p>
  );
});

const App: FC = () => {
  const [countStateA, setCountStateA] = useState<number>(0);
  const [countStateB, setCountStateB] = useState<number>(0);
  const countIncrementA = (): void => {
    setCountStateA(prevCountStateA => prevCountStateA + 1);
  };
  const countIncrementB = (): void => {
    setCountStateB(prevCountStateB => prevCountStateB + 1);
  };
  return (
    <div className="App">
      <CountResult text={"A button"} countState={countStateA} />
      <CountResult text={"B button"} countState={countStateB} />
      <button onClick={countIncrementA}>A button</button>
      <button onClick={countIncrementB}>B button</button>
    </div>
  );
};

export default App;
```

***

### useCallback

`memo`関数内でコールバック関数を使用したい場合は`useCallback`を使う。
つまり、`memo`と`useCallback`はニコイチの関係性である（`useCallback`から見たら）。
`useCallback`単体では何の役にも立たないので注意。

また、`useEffect`のように依存配列を渡せる。
そこにはコールバック関数が依存している要素を格納する。
そうすれば依存配列のいずれかの要素に変化があった場合のみ関数を作り直せる。

```javascript:title=JavaScript
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```

以下が具体的な例。

```typescript{19-20}:title=App.tsx
import "./App.css";
import { FC, memo, useState, useCallback } from "react";

type ButtonProps = {
  counterState: VoidFunction;
  buttonValue: string;
}

const Button: FC<ButtonProps> = memo(
  ({ counterState, buttonValue }: { counterState: VoidFunction; buttonValue: string }) => {
    console.log(`The ${buttonValue} button has been clicked.`);
    return <button onClick={counterState}>{buttonValue}</button>;
  }
);

const App: FC = () => {
  const [countStateA, setCountStateA] = useState<number>(0);
  const [countStateB, setCountStateB] = useState<number>(0);
  const countIncrementA = useCallback((): void => setCountStateA(countStateA + 1), [countStateA]);
  const countIncrementB = useCallback((): void => setCountStateB(countStateB + 1), [countStateB]);
  return (
    <div className="App">
      <p>A button: {countStateA}</p>
      <p>B button: {countStateB}</p>
      <Button counterState={countIncrementA} buttonValue="A button" />
      <Button counterState={countIncrementB} buttonValue="B button" />
    </div>
  );
};

export default App;
```

***

### useMemo

`useMemo`は関数の結果を保持するために使用する。
ちょっとパフォーマンス改善系のHooksはややこしいのでいったん整理する。

|            |      memo     | useCallback | useMemo  |
|    ----    |      ----     |    ----     |   ----   |
| メモ化の対象 | レンダリング結果 |   関数自体   | 関数の結果 |

何度計算しても同じ結果になる関数であれば`useMemo`を使ったほうが良い。
また、依存配列を使うことで要素のいずれかが変化した場合にのみメモ化された値を再計算する。

> 将来的に React は、例えば画面外のコンポーネント用のメモリを解放するため、などの理由で、メモ化された値を「忘れる」ようにする可能性があります。

公式ドキュメントには上記のような記載がある。
つまり、`useMemo`に依存した関数を書いてはいけない。
あくまでも、パフォーマンス最適化のために`useMemo`を使うべきであり、いつ廃止されても良い前提を持つ。
 
以下は実際の使用例。
`useMemo`の威力を確かめるにはあえて重い処理を書いたほうが分かりやすい。

```typescript{25}:title=App.tsx
import "./App.css";
import React, { useState, useMemo, FC } from "react";

const square = (params: number): number => {
  const testData: number[] = [...Array(30000).keys()];
  testData.forEach(() => {
    console.log(`Execute square function
    Loop processing is being executed ${testData.length} times.
    `);
  });
  return params * params;
};

const App: FC = () => {
  const [countStateA, setCountStateA] = useState<number>(0);
  const [countStateB, setCountStateB] = useState<number>(0);
  const countResultA = (): void => {
    setCountStateA(prevCount => prevCount + 1);
    console.log('The "A" button has been clicked.');
  };
  const countResultB = (): void => {
    setCountStateB(prevCount => prevCount + 1);
    console.log('The "B" button has been clicked.');
  };
  const squareArea = useMemo((): number => square(countStateB), [countStateB]);
  return (
    <div className="App">
      <p>
        Calculation results for A: {countStateA}
        <button onClick={countResultA}>Calculate: A + 1</button>
      </p>
      <p>
        Calculation results for B: {countStateB}
        <button onClick={countResultB}>Calculate: B + 1</button>
      </p>
      <b>Area of a square</b>
      <p>Calculation results for B x Calculation results for B = {squareArea}</p>
    </div>
  );
};

export default App;
```

***

### useRef

フォームを実装するときなど、DOMに直接アクセスしたい場面も多い。
そんな時には`useRef`を使用する。

このHooksを使うことでDOMノードへの参照を保持できる。
保持された値は更新してもコンポーネントが再レンダリングされることはない。

値を保持するHooksには`useState`もあるが`useState`は値が更新されるとコンポーネントが再レンダリングされる。

> 本質的に useRef とは、書き換え可能な値を .current プロパティ内に保持することができる「箱」のようなものです。

公式ドキュメントには上記の記載があるが、これは分かりやすい例えだ。
`useRef`の引数に渡した値は`refObject`の`current`プロパティの値になる。

ちょっと分かりにくいので実際に確かめてみる。
まずは`useRef`を使った方法。

```typescript{5}:title=App.tsx
import "./App.css";
import { FC, useEffect, useRef, useState } from "react";

const App: FC = () => {
  const inputRefObj = useRef<HTMLInputElement>(null);
  const [text, setText] = useState<string>("");
  useEffect((): void => {
    console.log("It has been rendered.");
  });
  const handleClick = (): void => {
    setText(inputRefObj.current!.value);
  };
  const textReset = (): void => {
    setText("");
    inputRefObj.current!.value = "";
  };
  return (
    <div className="App">
      <input ref={inputRefObj} type="text" />
      <button onClick={handleClick}>set text</button>
      <button onClick={textReset}>reset</button>
      <p>text: {text}</p>
    </div>
  );
};

export default App;
```

次に`useState`を使った方法。

```typescript{5}:title=App.tsx
import "./App.css";
import { ChangeEvent, FC, useEffect, useState } from "react";

const App: FC = () => {
  const [inputVal, setInputVal] = useState<string>("");
  const [text, setText] = useState<string>("");
  useEffect((): void => {
    console.log("It has been rendered.");
  });
  const handleClick = (): void => {
    setText(inputVal);
  };
  const handleChange = (e: ChangeEvent<HTMLInputElement>): void => {
    setInputVal(e.target.value);
  };
  const textReset = (): void => {
    setText("");
    setInputVal("");
  };
  return (
    <div className="App">
      <input value={inputVal} onChange={handleChange} type="text" />
      <button onClick={handleClick}>set text</button>
      <button onClick={textReset}>reset</button>
      <p>text: {text}</p>
    </div>
  );
};

export default App;
```

見た目上は変わりないが、コンソールにログを出力することで違いを明確に理解できた。
`useRef`では`handleClick`を実行した段階でコンポーネントがレンダリングされる（`inputRefObject.current`は更新されている）。
一方で`useState`を使った場合は文字をタイプするたびにレンダリングされていることが分かる。

中身が変更になってもそのことを通知したくない場合は`useRef`を使ったほうが良い。

上記の`useRef`を使ったアプローチでは`ref`オブジェクトを使用してDOMノードのvalue属性を書き換えている。
しかし、それはイミュータブルでもなければ宣言型でもない。
DOMを介してデータにアクセスするコンポーネントを**制御されていないコンポーネント**と呼ぶ。

上記の例では`useState`を使ってReact側で制御するほうが良いアプローチである。
確かに、文字を打つたびに再描画されるのはパフォーマンス的に心配だが、そこまで気にしなくても良い。
Reactは不要な再描画を避けるために、重複した描画要求を調停する仕組みを内部に持っている。

***

### useContext

前述したが基本的に状態はルートコンポーネントで一元管理するべきである。
しかし、コンポーネントツリーのネストが深くなるとルートと末端のコンポーネントの間に自分が使いもしないプロパティを追加しなければならない。
これではコードが複雑になり、スケーラビリティが損なわれてしまう。
例えば、調布から新宿へ行くのに各駅停車で行くようなものだ。そこはできれば特急に乗りたい。

その特急の役目を果たすのが**Context**である。
コンテキストを利用するには**Context Provider**にデータを渡す。
そして、コンテキストからデータを呼び出すには**Context Consumer**を利用する。
さっきの例に当てはめると調布駅がContext Provider、新宿駅がContext Consumerということになる。

Context利用の流れは下記のようになる。

1. `createContext`関数でオブジェクト（`Provider`と`Consumer`を含む）を作成
2. 参照可能にしたいコンポーネントを`Provider`で囲む
3. `useContext`でコンテキストからデータを取り出す

```typescript{4,6}:title=App.js
import "./App.css";
import { createContext, FC, useContext } from "react";

const SampleContextObj = createContext<string | undefined>(undefined);
const ConsumerComponent: FC = () => {
  const msgText = useContext(SampleContextObj);
  console.log(msgText);
  return <p>{msgText}</p>;
};

const msg: string = "Good morning!";

const App: FC = () => {
  return (
    <div className="App">
      <SampleContextObj.Provider value={msg}>
        <ConsumerComponent />
      </SampleContextObj.Provider>
    </div>
  );
};

export default App;
```

下記例では`TxtProvider`（別ファイルに切り出した）に囲まれたコンポーネントであればどこからでも`value`を参照できることが分かる。
`TxtContext`を`export/import`している点に注意。

```typescript{7}:title=TxtProvider.tsx
import { createContext, VFC } from "react";

type TxtProviderProps = {
  children: JSX.Element;
};

export const TxtContext = createContext<string | undefined>(undefined);

const txt: string = "This is the text passed from the provider.";

export const TxtProvider: VFC<TxtProviderProps> = ({ children }) => {
  return <TxtContext.Provider value={txt}>{children}</TxtContext.Provider>;
};
```

```typescript:title=App.tsx
import { TxtProvider } from "./TxtProvider";
import { First } from "./First";
import "./App.css";
import { FC } from "react";

const App: FC = () => {
  return (
    <div className="App">
      <TxtProvider>
        <First />
      </TxtProvider>
    </div>
  );
};

export default App;
```

```typescript:title=First.tsx
import { Second } from "./Second";
import { FC } from "react";

export const First: FC = () => {
  return (
    <>
      <p>First component</p>
      <Second />
    </>
  );
};
```

```typescript:title=Second.tsx
import { Third } from "./Third";
import { FC } from "react";

export const Second: FC = () => {
  return (
    <>
      <p>Second component</p>
      <Third />
    </>
  );
};
```

```typescript{5}:title=Third.tsx
import { FC, useContext } from "react";
import { TxtContext } from "./TxtProvider";

export const Third: FC = () => {
  const txtData = useContext(TxtContext);
  console.log(txtData);
  return (
    <>
      <p>
        Third component: <b>{txtData}</b>
      </p>
    </>
  );
};
```

注意点として`useContext`を呼び出すコンポーネントはコンテクストの値が変化するたびに毎回再描画される。
そのため、再描画が高価である場合はメモ化を使って最適化したほうが良い。
基本的にはシンプルで変更頻度が少ない値を渡す場合や、アプリ内の一部でのみ利用するデータに向いている。

## さいごに

実際にコードを書きながらブラウザで動きを確認することで用途が理解できた。
メモ化のHooksはともかく`useState`や`useEffect`、`useRef`はバリバリ使っていけそう。
パフォーマンスを無視すれば大抵のアプリは作れると実感したので、あとは実務や趣味でトライアンドエラーしながら身につけていきたい。

今回はよく見る基本的なHooksを扱った。
ただ、これで全てではないので残りのHooksもまた記事にまとめて理解を深めたい。
