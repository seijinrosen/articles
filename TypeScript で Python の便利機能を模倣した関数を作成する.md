---
title: TypeScript で Python の便利機能を模倣した関数を作成する
tags: TypeScript Python 競技プログラミング AtCoder アルゴリズム
author: seijinrosen
slide: false
---
普段は Python (PyPy) で競プロしていますが、AtCoder を TypeScript で解いていくなかで、Python ではビルトインや標準ライブラリで提供されている機能が欲しい場面が多くありました。

だんだん TypeScript のテンプレートも育ってきたので、Python の便利機能（関数やコンテナ）を模倣した TypeScript の自作関数をまとめて紹介します。

`TypeScript (3.8)` （AtCoder の 2022/08/22 現在のバージョン）で動作することを確認済みです。

なお、Python の `input()` を模倣したものは以下の記事に書きました。

https://qiita.com/seijinrosen/items/5a3c54d574d9622cd2ce

## max(), min()

```typescript:main.ts
const max = (data: number[]) => data.reduce((a, b) => Math.max(a, b));
const min = (data: number[]) => data.reduce((a, b) => Math.min(a, b));
```

配列の要素数が少なければ、スプレッド構文を利用して `Math.max(...array)` でも良いです。

しかし以下のドキュメントの通り、JavaScript の関数に多すぎる引数（おおよそ数万個以上とのことです）を渡すのは危険です（実際にエラーで通らなかったことを経験しました）。

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Function/apply#apply_をビルトイン関数と共に利用する

以下のドキュメントの通り、配列の最大値や最小値を求める場合は `Array.reduce()` を使用した方法が安全です。

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Math/max#配列の最大値の取得

なお、`console.log()` に配列のスプレッドを渡したときにも同様のエラーを経験しました。その場合は `Array.join(" ")` で文字列に変換してから出力するのがシンプルで良いと思います。

## sum(), statistics.mean()

```typescript:main.ts
const sum = (data: number[]) => data.reduce((a, b) => a + b);
const mean = (data: number[]) => sum(data) / data.length;
```

## zip() ＊配列2個まで

```typescript:main.ts
const zip = <T1, T2>(a: T1[], b: T2[]): [T1, T2][] =>
  [...Array(Math.min(a.length, b.length))].map((_, i) => [a[i], b[i]]);
```

配列3個以上への対応や、`longest = true` とすることで `itertools.zip_longest` と同等の動きを実現するアイデアがあります。今後問題を解いていく中で必要になったら実装してみます。

## str.startswith(), str.endswith()

```typescript:main.ts
const startsWith = (s: string, prefixArray: string[]) =>
  prefixArray.some((prefix) => s.startsWith(prefix));
const endsWith = (s: string, suffixArray: string[]) =>
  suffixArray.some((suffix) => s.endsWith(suffix));
```

## bisect.bisect_right()

```typescript:main.ts
const bisectRight = (array: number[], x: number) => {
  let lo = 0;
  let hi = array.length;
  while (lo < hi) {
    const mid = Math.floor((lo + hi) / 2);
    x < array[mid] ? (hi = mid) : (lo = mid + 1);
  }
  return lo;
};
```

二分探索の便利関数です。

以下の Python のソースコードを参考にしました。

https://github.com/python/cpython/blob/4f100fe9f1c691145e3fa959ef324646e303cdf3/Lib/bisect.py#L15-L34

実戦で必要になったら、`lo`, `hi` を引数として受け入れるようにしたり、`bisect_left()` などの他の `bisect` モジュールの関数も追加したりしようと考えています。

## collections.Counter()

```typescript:main.ts
const Counter = <T>(arr: T[]) => {
  const counter = new Map<T, number>();
  for (const v of arr) counter.set(v, (counter.get(v) || 0) + 1);
  return counter;
};
```

`Map` を返す関数として実装しています。`most_common` などのメソッドに相当するものはその場で書いてしまっています。

関数名を大文字で始めるのは好き嫌いがありそうですが、`const counter = Counter(array);` とすることで、Python との頭の切り替えを少なくするためにこうしています。

## itertools.accumulate()

```typescript:main.ts
const accumulate = (data: number[]) => {
  let now = 0;
  return data.map((v) => (now += v));
};
```

累積和の最初にゼロを入れるために、`const acc = [0, ...accumulate(arr)];` として使用することが多いです。

## range()

```typescript:main.ts
const range = (start: number, stop: number, step = 1) =>
  Array.from({ length: (stop - start) / step + 1 }, (_, i) => start + i * step);
```

以下の MDN ドキュメントの `range()` 関数に、型とデフォルト値を追加したものです。

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/from#連番の生成_範囲指定

:::note warn
注意点として、Python の `range()` とは異なり `stop` の値も終端として含まれます。
:::

## string.ascii_letters

```typescript:main.ts
const range = (start: number, stop: number, step = 1) =>
  Array.from({ length: (stop - start) / step + 1 }, (_, i) => start + i * step);
const alphaArray = (start: string, stop: string) =>
  range(start.charCodeAt(0), stop.charCodeAt(0)).map((x) =>
    String.fromCharCode(x)
  );
```

Python の `string.ascii_letters` とはだいぶ趣が異なりますが、それなりに使いやすいのではないかと思います。

`range()` 同様、MDN の以下のドキュメントを参考にしました。

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/from#連番の生成_範囲指定

## set() & set()

```typescript:main.ts
const intersection = <T>(setA: Set<T>, setB: Set<T>) => {
  const _intersection = new Set();
  for (const elem of setB) if (setA.has(elem)) _intersection.add(elem);
  return _intersection;
};
```

MDN のドキュメントの関数に型をつけたものです。個人的によく使うので、テンプレートに入れています。

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Set#基本的な集合演算の実装

## 終わりに

テンプレートを更新したら、追記・更新します。
