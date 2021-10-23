---
title: "関数型言語としてのJavaScript"
date: "2021-10-17T15:31:56.421Z"
category: "t"
description: ""
emoji: "💡"
slug: "4343468"
published: true
---

## モチベーション

最近はReactを触る機会が多く、関連する書籍を読み漁っている。 そこでJavaScriptを関数型で書くという面白いアプローチを見つけたので纏めてみようと思う。

因みに私は普段JavaやRubyなどの言語を使用しているがいずれも命令型プログラミングとして記述している。 宣言型プログラミングのパラダイムには以前から興味があり、Scalaを少し触ったことがあるけど正直あまり理解できていない。

今回参考にした書籍はオライリー・ジャパン社から出版された「[Reactハンズオンラーニング 第2版](https://www.amazon.co.jp/React%E3%83%8F%E3%83%B3%E3%82%BA%E3%82%AA%E3%83%B3%E3%83%A9%E3%83%BC%E3%83%8B%E3%83%B3%E3%82%B0-%E7%AC%AC2%E7%89%88-%E2%80%95Web%E3%82%A2%E3%83%97%E3%83%AA%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E9%96%8B%E7%99%BA%E3%81%AE%E3%83%99%E3%82%B9%E3%83%88%E3%83%97%E3%83%A9%E3%82%AF%E3%83%86%E3%82%A3%E3%82%B9-Alex-Banks/dp/4873119383)」である。
コードの大部分を使うのでなければサンプルコードを流用しても良いと書かれているので実際に動かしながら試してみる。

## そもそも関数型とは

関数型プログラミングとは**第一級オブジェクト**（変数に格納できるだけではなく、関数の引数として渡したり、戻り値として受け取れるオブジェクト）を用いて関数を主軸にコードを組み立てるスタイルのことである。

また、関数型プログラミングは宣言型プログラミングという大きなプログラミングパラダイムの一部であり、その反対には命令型プログラミングが存在する。

[[i | 簡単な違い]]
| * 命令型プログラミング
|   * 「何をするのか」（what）を重視
|   * コンピュータが読みやすい?
|   * 文を重視
| * 宣言型プログラミング
|   * 「どのようにするのか」（how）を重視
|   * 人間が読みやすい
|   * 式を重視

これだけでは分かりにくいので実際にコードを書いて比較してみる。以下は英文中の空白をハイフンに変換するプログラムである。
実行環境はGASです（小さなプロジェクトとして始めやすいから）。

まずは命令型。

```javascript:title=main.gs
function imperative() {
  let string = "this is a pen";
  let urlFriendly = "";

  for (let i = 0; i < string.length; i++) {
    if (string[i] === " ") {
      urlFriendly += "-";
    } else {
      urlFriendly += string[i];
    }
  }

  console.log(urlFriendly); // this-is-a-pen
}
```

こちらの方が個人的には馴染み深い。

次に宣言型。

```javascript:title=main.gs
function declarative() {
  const string = "this is a pen";
  const urlFriendly = string.replace(/ /g, "-");

  console.log(urlFriendly); // this-is-a-pen
}
```

命令型の方が宣言型に比べて簡潔にかけており、中身を見ても非エンジニアでも理解しやすいコードになっている。
それはつまり、実装に関する詳細は個々の関数により**抽象化され隠蔽されている**ということだ。
ちなみに実行速度は命令型の方が早いが何万回も関数を実行するのでなければ気にするほどではない。

## 関数型プログラミングの基本

関数型プログラミングを形成している概念に以下の5つがある。

1. イミュータブルなデータ
2. 純粋関数
3. データの変換
4. 高階関数
5. 再帰

それぞれ例を交えつつ解説していく。

***

### イミュータブルなデータ

イミュータブルとは**変更を加えることができない状態**を指す。
関数型プログラミングでは全てのデータはイミュータブルであり、書き換えはできない。

書き換えができないのは大きな制限に感じるかもしれないが、単純にオブジェクトのコピーを作成してから変更すれば良いだけである。
例として以下のオブジェクトを変更してみる。

```javascript:title=main.gs
let color_lawn = {
  title: "lawn",
  color: "#00ff00",
  rating: 0
};
```

まずは元のオブジェクトを変更してしまった例。

```javascript:title=main.gs
const rateColor = (color, rating) => {
  color.rating = rating;
  return color;
}

console.log(rateColor(color_lawn, 5).rating); // 5
console.log(color_lawn.rating); // 5
```

JavaScriptで関数の引数としてオブジェクトを受け取った場合、それはコピーではなく、実際のオブジェクトへの参照となる。
つまり、オブジェクトのプロパティの値を変更することは元のオブジェクトを変更するということだ。

次にイミュータブルな例。

```javascript:title=main.gs
const rateColor = (color, rating) => ({
  ...color,
  rating
});

console.log(rateColor(color_lawn, 5).rating); // 5
console.log(color_lawn.rating); // 0
```

アロー関数とスプレッド構文を使用している。
スプレッド構文を使用することで簡潔にオブジェクトのコピーを作成できるので積極的に使っていきたい。

もう1つ、配列をイミュータブルに扱う例を見てみる。
以下がその対象の配列である。

```javascript:title=main.gs
let list = [{ title: "Rad Red" }, { title: "Lawn" }, { title: "Party Pink" }];
```

これに要素を追加してみる。まずは元の配列を変更してしまった例。

```javascript:title=main.gs
const addColor = (title, colors) => {
  colors.push({ title: title });
  return colors;
};

console.log(addColor("Glam Green", list).length); // 4
console.log(list.length); // 4
```

`Array.push`は破壊的なメソッドなので関数型プログラミングにおいては使用しないようにする。代わりに元の配列を変更しない`Array.concat`を使用する。
また、スプレッド構文を使用することで下記のように簡潔に書くこともできる。

```javascript:title=main.gs
const addColor = (title, list) => [...list, { title }];

console.log(addColor("Glam Green", list).length); // 4
console.log(list.length); // 3
```

こちらは配列のコピーを作成後に新しい要素を追加し、コピーした方を戻り値として返却している。

***

### 純粋関数

[[i | 純粋関数の定義]]
| * 引数の値のみを参照して、それを元に計算してから値を返す関数
| * 少なくとも1つの引数を取り、値 or 他の関数を戻り値として返却
| * 副作用がない
|   * グローバル関数に値を書き込まない
|   * アプリケーションの状態を変更しない
| * 引数をイミュータブルなデータとして扱う

これだけでは分かりにくいので実際にコードを書いてみる。
まずは純粋ではない関数から。

```javascript:title=main.gs
const ktnkk = {
  name: "Keiten Kiki",
  canRead: false,
  canWrite: false
};

const selfEducate = () => {
  ktnkk.canRead = true;
  ktnkk.canWrite = true;
  return ktnkk;
};

selfEducate();
console.log(ktnkk); // { name: 'Keiten Kiki', canRead: true, canWrite: true }
```

これが純粋関数ではない理由は以下の通り。

* 関数が1つも引数を取らず、戻り値を返却しない
* 副作用がある
  * 関数外で宣言された変数に変更を加えているから

このような関数を実行してしまうと環境が汚染されてしまうので良くない。
以下のような純粋関数を使用することで保守性の高いコードを書くことができる。

```javascript:title=main.gs
const ktnkk = {
  name: "Keiten Kiki",
  canRead: false,
  canWrite: false
};

const selfEducate = person => ({
  ...person,
  canRead: true,
  canWrite: true
});

console.log(selfEducate(ktnkk)); // { name: 'Keiten Kiki', canRead: true, canWrite: true }
console.log(ktnkk); // { name: 'Keiten Kiki', canRead: false, canWrite: false }
```

純粋関数は関数型プログラミングの肝となる部分である。

[[i | 関数を書く際のポイント]]
| * 少なくとも1つの引数を受け取る
|   * 引数以外の値を参照してはいけない（グローバル変数の値に左右されない）
| * 値 or 他の関数を戻り値として返却する
|   * 同じ引数であれば必ず同じ戻り値を返却する（**参照透過性**）
| * 引数や関数外で定義された変数に直接変更を加えない

***

### データの変換

上記2つの概念から関数型プログラミングにおけるデータの変換は、まず第一にコピーをしてからそれを別の関数に渡して値を書き換える手順を踏む必要があると分かった。
そのようなデータ変換にはJavaScriptのビルトイン関数で対応する。

その際に大事なことは非破壊的メソッドを使用することである。
特に大事な3つのメソッドは実際に挙動を確認してみる。

1. `Array.filter`
2. `Array.map`
3. `Array.reduce`

#### Array.filter

```javascript:title=main.gs
const shikoku = ["Kochi", "Ehime", "Kagawa", "Tokushima"];

const kPref = shikoku.filter(pref => pref[0] === "K");
const cutPref = (cut, shikoku) => shikoku.filter(pref => pref !== cut);

console.log(kPref.join(", ")); // Kochi, Kagawa
console.log(cutPref("Kochi", shikoku).join(", ")); // Ehime, Kagawa, Tokushima
console.log(shikoku.join(", ")); // Kochi, Ehime, Kagawa, Tokushima
```

`kPref`では頭文字がKの文字列を取得している。
このように配列から他の要素を削除する場合は`Array.pop`や`Array.splice`（破壊的メソッド）ではなく、`Array.filter`を用いる。

関数`cutPref`は第2引数に入れられた文字列を削除する純粋関数である。

#### Array.map

```javascript:title=main.gs
const oldShikoku = [{ name: "Tosa" }, { name: "Iyo" }, { name: "Sanuki" }, { name: "Awa" }];

const updatePref = (oldName, name, arr) => arr.map(pref => (pref.name === oldName ? { ...pref, name } : pref));

const currentPref = updatePref("Tosa", "Kochi", oldShikoku);

console.log(currentPref[0]); // { name: 'Kochi' }
console.log(oldShikoku[0]); // { name: 'Tosa' }
```

純粋関数`updatePref`は配列`shikoku`を受け取り、コピーした後に新しい配列として返す（配列 -> 配列）。
三項演算子を使うことで簡潔に書いているがちょっと見にくいので無理に使わなくても良いかもしれない。

`Array.map`は実行後の結果を配列で返してくれるのでかなり使い勝手が良く、頻出なのでマスターしなければならない。
`forEach`との違いは新しい配列を返すかどうかで、関数型プログラミングではあまり使わない。

#### Array.reduce

```javascript:title=main.gs
const randomNums = [38, 11, 29, 28, 81, 99, 41, 57];

const max = nums => nums.reduce((max, value) => (value > max ? value : max), 0);

console.log(max(randomNums)); // 99
```

見ての通りだが、純粋関数`max`は配列内の整数から最大値を返してくれる。
`Array.reduce`は配列から単一の値へ変換する際に使用する。`Array.reduceRight`というビルトイン関数もあるけどそれは配列を末尾から走査して値を集約する。

この使い方の他にも`Array.reduce`は既存の配列を異なる配列に変換することもできる。
以下がその例。

```javascript:title=main.gs
const randomNums = [38, 11, 38, 28, 11, 99, 99, 28];

const uniqueNums = nums => nums.reduce((unique, num) => (unique.indexOf(num) !== -1 ? unique : [...unique, num]), []);

console.log(uniqueNums(randomNums)); // [ 38, 11, 28, 99 ]
console.log(randomNums); // [ 38, 11, 38, 28, 11, 99, 99, 28 ]
```

純粋関数`uniqueNums`は値が重複している配列からユニークな値による配列に変換してくれる。
この時、`Array.reduce`の第2引数には配列の初期値として空の配列`[]`を渡している点に注意。
この配列に対してコールバック関数内で値が次々に追加されることになる。

***

### 高階関数

高階関数とは他の関数を引数に取るか、戻り値として関数を返すか、もしくはそれら両方を満たす関数のことである。
想像がつきにくいかもしれないけど、データの変換の時に使用した`Array.filter`、`Array.map`、`Array.reduce`は関数を引数に取るため高階関数ということになる。

実際に自分で実装してみると以下のような形になる。

```javascript:title=main.gs
const greetIf = (isMorning, fnTrue, fnFalse) => (isMorning ? fnTrue() : fnFalse());

const sayGoodMorning = () => console.log("Good morning!");
const sayHello = () => console.log("Hello!");

greetIf(true, sayGoodMorning, sayHello); // Good morning!
greetIf(false, sayGoodMorning, sayHello); // Hello!
```

挨拶を返す高階関数`greetIf`は`fnTrue`と`fnFalse`の2つのコールバック関数を引数に取る。
そして、引数`isMorning`の値次第でいづれかのコールバック関数を呼び出す。

上記では引数に関数を使用したが、戻り値として関数を返す高階関数は非同期処理をシンプルに書くために使われることが多い。
つまり、その場で実行せずに関数として返却することで処理の実行タイミングを変えられる訳であるが、その際には**カリー化**というテクニックを使う。

```javascript:title=main.gs
// 非同期関数getFakeMembersは省略

const userLogs = userName => message => console.log(`${userName} -> ${message}`);

console.log(userLogs("ktnkk"));

log("attempted to load 20 fake members");
getFakeMembers(20)
  .then(members => log(`successfully loaded ${members.length} members`))
  .catch(error => log("encountered an error loading members"));

// ktnkk -> attempted to load 20 fake members
// ktnkk -> successfully loaded 20 members

// ktnkk -> attempted to load 20 fake members
// ktnkk -> encountered an error loading members
```

関数`userLogs`は`userName`を引数に取るが、必要な情報が全て揃っていないため関数を戻り値として返却する。
その後、必要な情報が入手できた時点で再び関数を呼び出すことで出力が完了する。
`=>`が2つあるのが特徴的。

***

### 再帰

関数の中から自分自身を再起的に呼び出すテクニックのことを**再帰**という。
ループ系のコードを再帰を用いることで簡潔に記述することができる。以下がその例。

```javascript:title=main.js
const countdown = (val, fn, delay = 1000) => {
  fn(val);
  return val > 0 ? setTimeout(() => countdown(val - 1, fn, delay), delay) : val;
};

const log = val => console.log(val);
countdown(5, log);

// 5
// 4
// 3
// 2
// 1
// 0
```

※ GASでは`setTimeout()`は使えなかった。

再帰は非同期関数との相性が抜群なので使える場面では必ず使いたい。
そして、非同期関数と同じくらい相性が良い用途はデータ構造の探索である。

例として以下のネストしたオブジェクトから必要なプロパティ値を取得する再帰関数を作成する。

```javascript:title=main.js
const ktnkk = {
  type: "human",
  data: {
    gender: "secret",
    info: {
      id: 99,
      fullName: {
        first: "Keiten",
        last: "Kiki"
      }
    }
  }
};

const deepPick = (fields, object = {}) => {
  const [first, ...remaining] = fields.split(".");
  return remaining.length ? deepPick(remaining.join("."), object[first]) : object[first];
};

deepPick("type", ktnkk); // "human"
deepPick("data.info.fullName.last", ktnkk); // "Keiten"
```

※ このコードもGASでは使えなかった。どうも`split`で引っ掛かる。

[[i | 再帰関数 deepPick]]
| 1. プロパティ名をドット演算子で先頭と後続に分割
| 2. 後続の要素が存在しなければ先頭の要素に呼応するプロパティ値を返却
| 3. 後続の要素が存在すれば自信を再帰呼び出し
| 4. オブジェクトの一段深いところに潜る
| 5. プロパティ名がドット演算子を含まなくなるまで繰り返す

## さいごに

ざっと関数型言語の特徴を抑えてみたが、実際にこれらの関数を合成させることで1つのアプリケーションが構築される。
つまり、更に大きな関数を作っていく訳だが、その工程がとても面白いと思う。

命令型プログラミングに慣れているので宣言型で書くのにはちょっと時間が掛かるけど近年の開発では関数型が主流になりつつあるように感じているし、今までネックだったバグの特定やテストコードの複雑さを解決する手段としてはベストだと思うので必ず習得したい。
いきなり業務で使うと周りも混乱すると思うからまずは小さなプログラム（例えばGAS）から関数型で書くようにしてみようかな。
ゆくゆくはHaskellみたいな純粋関数型プログラミング言語も書いてみたい。ただ、現実的（業務で使えそう）なのは使い慣れたJavaに似ているScalaかも。

いずれにしても、関数型の概念を手続き型に応用することで副作用のないコードを書く手助けにはなると思うから学習して無駄になることはないだろう。
