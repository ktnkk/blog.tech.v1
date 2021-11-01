---
title: "React Hooksの初歩"
date: "2021-10-18T15:31:56.421Z"
category: "t"
description: ""
emoji: "🪝"
slug: "2957598"
published: false
---

## プロローグ

React Hooksという状態管理をするための機能があるのだが、今まで何となく触ってきたのでちゃんと理解したいと思う。
ブログ程度であればなくても困らなかったけど、大規模なアプリ開発には必須だし、ちょっと小洒落たサイトを作る際も必要不可欠だ。

ざっと見た感じだと上手く使えればパフォーマンスの向上も期待できる。
出来ればこのブログにも今後取り入れてチューニングしたい。

目標としては新規開発でHooksを使う前提の設計ができるようになることと、既存コードの改修や機能追加（このパターンが多い）でコードの意図を適切に理解できること。

もちろん、全てのHooksを習得すべきであるとは思うが、全く見たことがないものや使う頻度の低いものよりも使う確率が高いものから習得したほうが効率は良い。
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


