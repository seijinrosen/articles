---
title: 【AtCoder】TypeScript で Python の `input()` ライクな入力を実現する
tags: AtCoder TypeScript Python 競技プログラミング 競プロ
author: seijinrosen
slide: false
---
Python の `input()` のように、入力を1行ずつ読み込むような関数を実現する TypeScript のテンプレートを作成しました。

```typescript:main.ts
import { readFileSync } from "fs";
let inputLinesIndex = 0;
const inputLines = readFileSync("/dev/stdin", "utf8").split("\n");
const input = () => inputLines[inputLinesIndex++];

// おまけ
// Python の `print()` に似た関数
const print = (...messages: any) => console.log(...messages);
```

`input()` を呼び出すたびに `inputLinesIndex` がインクリメントし、結果的に1行ずつ読み込まれるような挙動を実現できます。

`TypeScript (3.8)` （AtCoder の 2022/08/21 現在のバージョン）で動作することを確認済みです。

追記：`input()` 以外の Python ライクな機能も TypeScript で実装した、という記事を書きました。

https://qiita.com/seijinrosen/items/3b99659e0434df09422f

以下、Python での例と並べて代表的な使用例を記載します。

## 使用例

### 文字列1行

$S$

例：[ABC 260 A - A Unique Letter](https://atcoder.jp/contests/abc260/tasks/abc260_a)

文字列として1行読み込むだけ。

```python:main.py
S = input()
```

```typescript:main.ts
import { readFileSync } from "fs";
let inputLinesIndex = 0;
const inputLines = readFileSync("/dev/stdin", "utf8").split("\n");
const input = () => inputLines[inputLinesIndex++];

const S = input();
```

### 整数1個

$N$

例：[ABC 256 A - 2^N](https://atcoder.jp/contests/abc256/tasks/abc256_a)

整数を1つだけ読み込む。

```python:main.py
N = int(input())
```

```typescript:main.ts
import { readFileSync } from "fs";
let inputLinesIndex = 0;
const inputLines = readFileSync("/dev/stdin", "utf8").split("\n");
const input = () => inputLines[inputLinesIndex++];

const N = +input();
// 下記と同等です。
// const N = Number(input());
```

`+`をつけるだけで `number` 型になります。

### 整数2個以上

$L_1\ R_1\ L_2\ R_2$

例：[ABC 261 A - Intersection](https://atcoder.jp/contests/abc261/tasks/abc261_a)

1行に2個以上の場合は [分割代入](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) を使うと便利です。

```python:main.py
L1, R1, L2, R2 = map(int, input().split())
```

```typescript:main.ts
import { readFileSync } from "fs";
let inputLinesIndex = 0;
const inputLines = readFileSync("/dev/stdin", "utf8").split("\n");
const input = () => inputLines[inputLinesIndex++];

const [L1, R1, L2, R2] = input().split(" ").map(Number);
```

`.map(Number)` をつけなければ `string` 型として代入されます。

### 配列（リスト）

$N$  
$A_1\ A_2\ \dots\ A_N$

例：[ABC 256 B - Batters](https://atcoder.jp/contests/abc256/tasks/abc256_b)

```python:main.py
N = int(input())
A = list(map(int, input().split()))
# 内包表記でもOK
# A = [int(x) for x in input().split()]
```

```typescript:main.ts
import { readFileSync } from "fs";
let inputLinesIndex = 0;
const inputLines = readFileSync("/dev/stdin", "utf8").split("\n");
const input = () => inputLines[inputLinesIndex++];

const N = +input();
const A = input().split(" ").map(Number);
```

### 縦方向の配列

$N$  
$W_1$  
$W_2$  
$\vdots$  
$W_N$

例：[ABC 109 B - Shiritori](https://atcoder.jp/contests/abc109/tasks/abc109_b)

```python:main.py
N = int(input())
W = [input() for _ in range(N)]
```

```typescript:main.ts
import { readFileSync } from "fs";
let inputLinesIndex = 0;
const inputLines = readFileSync("/dev/stdin", "utf8").split("\n");
const input = () => inputLines[inputLinesIndex++];

const N = +input();
const W = [...Array(N)].map(() => input());
```

$W_i$ が数値なら `input()` を `+input()` にします。

### タプルの配列

$N$  
$A_1\ B_1$  
$\vdots$  
$A_N\ B_N$

例：[ABC 181 B - Trapezoid Sum](https://atcoder.jp/contests/abc181/tasks/abc181_b)

```python:main.py
N = int(input())
AB: "list[tuple[int, int]]" = [tuple(map(int, input().split())) for _ in range(N)]
```

```typescript:main.ts
import { readFileSync } from "fs";
let inputLinesIndex = 0;
const inputLines = readFileSync("/dev/stdin", "utf8").split("\n");
const input = () => inputLines[inputLinesIndex++];

const N = +input();
const AB: [number, number][] = [...Array(N)]
  .map(() => input().split(" "))
  .map(([a, b]) => [+a, +b]);

// 関数化したもの
const inputPairArray = (n: number): [number, number][] =>
  [...Array(n)].map(() => input().split(" ")).map(([a, b]) => [+a, +b]);
// const AB = inputPairArray(N);
```

型を適切につけるためにちょっと面倒なことをしています（もっとシンプルに書ける方法もあるかもしれません）。

型が `number[][]` で構わない場合や、そもそもタプルの配列ではなく `配列A` と `配列B` で別々に管理する場合は、異なる書き方になるでしょう。

私はなるべく `for` 文を使わない方法（関数型っぽい方法？）を好んでいます。

#### 2022/09/09 追記

型アサーションを使った方法を思いつきました。型アサーションの使用を許容できれば、こちらのほうがシンプルで拡張性が高いので良いと思います。

```typescript:main.ts
import { readFileSync } from "fs";
let inputLinesIndex = 0;
const inputLines = readFileSync("/dev/stdin", "utf8").split("\n");
const input = () => inputLines[inputLinesIndex++];

const N = +input();
const AB = [...Array(N)].map(
  () => input().split(" ").map(Number) as [number, number]
);

// 関数化したもの
const inputPairArray = (n: number) =>
  [...Array(n)].map(() => input().split(" ").map(Number) as [number, number]);
// const AB = inputPairArray(N);
```

## 終わりに

今まで競プロではほぼ Python しか使ってきませんでしたが、TypeScript も結構書き心地が良いです。

Python と比べて、

- 配列のメソッドチェーン（`map`、`filter` など）の最中に一時変数をつくることができる
  - Python で同等のことはおそらく不可能
- きちんと型のついた小さい関数をさらっと作れる
  - Python だとトップレベルのラムダ式に型をつけるのは面倒
  - 普通に `def` で定義するのはおおげさな場合がある

などの点で優位性を感じます。

とはいえ、Python の内包表記や、`itertools` や `collections` などの標準ライブラリの機能は非常に強力なので、本番ではこれからも基本的には Python（というか PyPy）を使っていくでしょう（C++ はいつまで経っても慣れないので……）。

新しい入力パターンに出くわしたら、またこの記事に追記していきます。
