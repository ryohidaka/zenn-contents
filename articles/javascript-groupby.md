---
title: "JavaScriptにgroupByが実装されたから、Lodashの.groupByと比較してみた"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "lodash"]
published: true
---

## TL;DR

- JavaScript 標準に`groupBy`関数が実装されました。
- 2023 年 12 月時点で、Safari など一部のブラウザは未対応です。
- [Underscore.js](https://underscorejs.org/) や [Lodash](https://lodash.com/) の`.groupBy()`と同等のコード量と可読性を、ネイティブのみで実現できます。

## はじめに

### JavaScript 標準に groupBy() が実装されたらしい

プログラミング言語やデータベースクエリでよく使われる `groupBy` は、特定の列やキーを指定してデータをグループにまとめる機能です。Python などでは既に標準機能として搭載されています。

https://qiita.com/silane1001/items/5f2a4f06e964e7443433

https://dev.to/shameel/javascripts-grouping-methods-objectgroupby-and-mapgroupby-aba

これまでは JavaScript で `groupBy` と同様の処理を実装するには、以下の方法がありました：

- [Underscore.js](https://underscorejs.org/) や [Lodash](https://lodash.com/) などのライブラリを使用する
- `reduce()`など、標準で実装された関数を利用する

### Underscore.js とは

> Underscore.js は JavaScript で使用されるライブラリの一種です。便利な関数を 100 以上も用意していることで人気を集めております。

https://anken-hyouban.com/blog/2021/09/17/underscore-js/

### Lodash とは

> Lodash は、軽量な JavaScript ユーティリティライブラリです。配列やオブジェクト、文字列などに対して頻用される処理がモジュールとして用意されています。

https://dev.classmethod.jp/articles/nodejs-array-manipulation-is-useful-if-you-master-lodash/

## 実際に比べてみた

### Lodash のサンプルコード

[Lodash のドキュメント](https://lodash.com/docs/4.17.15#groupBy)では以下の二つの実装例が紹介されています。
今回はこのサンプルコードをもとに比較します。

```js
// Lodash
var grouped = _.groupBy([6.1, 4.2, 6.3], Math.floor);
console.log(grouped);
// => { '4': [4.2], '6': [6.1, 6.3] }

// The `_.property` iteratee shorthand.
var grouped = _.groupBy(['one', 'two', 'three'], 'length');
console.log(grouped);
// => { '3': ['one', 'two'], '5': ['three'] }
```

### これまでの代替実装

これまでは `reduce()` を用いて以下のように実装する必要がありました。

https://github.com/you-dont-need/You-Dont-Need-Lodash-Underscore#_groupby

```js
// Native with reduce()
var grouped = [6.1, 4.2, 6.3].reduce(
  (r, v, i, a, k = Math.floor(v)) => ((r[k] || (r[k] = [])).push(v), r),
  {}
);
console.log(grouped);
// => { '4': [4.2], '6': [6.1, 6.3] }

// Native with reduce()
var grouped = ['one', 'two', 'three'].reduce(
  (r, v, i, a, k = v.length) => ((r[k] || (r[k] = [])).push(v), r),
  {}
);
console.log(grouped);
// => { '3': ['one', 'two'], '5': ['three'] }
```

### Object.groupBy()を用いた代替実装

`Object.groupBy()`を用いることで、lodash とほぼ同等の記述量で実装できるようになりました。
また、引数が現在の要素を示す`v`のみなので、可読性も向上しています。

```js
// Native with Object.groupBy()
var grouped = Object.groupBy([6.1, 4.2, 6.3], (num) => Math.floor(num));
console.log(grouped);
// => { '4': [4.2], '6': [6.1, 6.3] }

// Native with Object.groupBy()
var grouped = Object.groupBy(['one', 'two', 'three'], (v) => v.length);
console.log(grouped);
// => { '3': ['one', 'two'], '5': ['three'] }
```

## まとめ

今更感はありますが、コード量の削減と可読性の向上が期待できるので、個人的には嬉しい機能追加だと思います。
今後、Safari でも正式サポートされることを期待しています。
