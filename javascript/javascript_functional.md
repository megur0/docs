---
title: "配列操作・アロー関数など - JavaScript"
updated: 2026-07-24
---

[TOP(About this memo))](../README.md) > [一覧(JavaScript)](./README.md) > 配列操作・アロー関数など


## forEachやmapではbreakできない
* for文で書いてbreakするしかない。


## map, forEach, filter, reduce
* 配列に対して使える。
```js
var array = [2, 3, 5];
var num = array.map(function(value) {
    // 配列の各値を3倍にする
    return value * 3;
});
console.log(num);

const array1 = ['a', 'b', 'c'];
array1.forEach(element => console.log(element));
// expected output: "a"
// expected output: "b"
// expected output: "c"

// 1 + 2 + 3 + 4
console.log([1, 2, 3, 4].reduce((previousValue, currentValue) => previousValue + currentValue));
// expected output: 10
```

### 実用例
```js
return Object
    .keys(this.$route.query)
    .filter(key => key !== "redirect_url")
    .map(key => key + "=" + this.$route.query[key])
    .join('&');
```
* ポイント
    * mapを使わない場合は、配列を用意してpushし、joinするという書き方になるため、どうしても複数文に分かれる。
    * JavaScriptのmapは配列にしか適用できないため、`keys()`で配列にしてから、mapの中でオブジェクトへキー経由でアクセスしている。
    * filterで`redirect_url`がキーの場合を除外している。
    * (IMO)(?) このやり方は、元のオブジェクトのキーの順番が保証されるとは限らない点に注意した方がよさそう。
* (参考)
    * https://qiita.com/TelBouzu/items/08ce089d2747d978fd0f
    * https://blog.ryo4004.net/web/3988/

### TODO
* reduceの使い所についてもう少し整理する。


## アロー関数
* ES6で使えるようになった。
```js
// 単一式の場合はブラケットやreturnを省略できる
const fn = (a, b) => a + b;

// ブラケットやreturnを省略してオブジェクトを返したい場合は`()`で囲む
const fn = (a, b) => ({ sum: a + b });
```
```js
(function(name) {
    console.log("Hello " + name)
})("taro");
// => "Hello taro"

// アロー関数を使うとこんな感じ
(name => console.log("Hello " + name))("taro");
// => "Hello taro"
```
* 従来の無名関数は関数の呼び方によって`this`の参照先が異なるが、アロー関数は関数が定義されたスコープ内の`this`を参照する。
    * (参考) https://qiita.com/soarflat/items/b251caf9cb59b72beb9b


## 即時関数
* 以下のように無名関数（クロージャ）と組み合わせて構造を隠蔽できる。
```js
var module = (function() {
    var count = 0;
    return {
        increment: function() {
            count++;
        },
        show: function() {
            console.log(count);
        }
    };
})();

module.show(); // 0
module.increment();
module.show(); // 1
console.log(count); // undefined
```


## ラムダ式
* (IMO)(?) 結局のところシンタックスシュガーのようなものと捉えている。
* (参考) http://softcommu-blog.com/?eid=9


## スプレッド構文
* 反復可能なオブジェクト（配列など）を、文脈に合わせて展開する。
* (参考) https://analogic.jp/spread-operator/
```js
var arr = [1, 2, 3];
console.log(arr); // [1, 2, 3]
console.log(...arr); // 1 2 3
```
