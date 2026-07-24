---
updated: 2026-07-24
---

[TOP(About this memo))](../README.md) > [一覧(JavaScript)](./README.md) > 関数オブジェクト・スコープ


# 関数はオブジェクト（WIP）
* 関数もオブジェクトである。ただし、関数はかなり特殊なオブジェクトである（配列よりもずっと特殊）。
* `()`をつけると呼び出せるというのは関数だけが持つ特徴。
* 以下の2つの書き方は同じ意味。
```js
function hello(message) {
    alert(message);
}
hello("word");

var hello = function(message) {
    alert(message);
}
hello("word");
```
* 実は以下のようにも書ける（関数にはコンストラクタが存在する）。
```js
var hello = new Function("message", "alert(message);");
hello("word");
```
* 例えば `alert(alert);` とやると `"function alert() { [native code] }"` のように出力される。
    * これはalertが、alertという名前の変数に入った関数だから。
    * `[native code]` は、この関数の実行がブラウザの内部処理で行われる（JavaScriptで書かれた定義がない）ことを意味する。
    * より詳しくは、[this・プロトタイプ・class](./javascript_this_prototype_class.md)を参照。

## 仮引数の長さ
* `hello.length` で参照可能。

## 可変長引数
* 関数を呼び出したときの引数の情報が入っている（配列風の）オブジェクト`arguments`を使えば実現可能。

## applyとcall
* `Function`のプロトタイプには以下の特別なメソッドがある。どちらも関数オブジェクト（`Function`で生成されたインスタンス）で利用可能。
    * `Function.prototype.apply(thisArg, argArray)`
    * `Function.prototype.call(thisArg[, arg1, ...])`
* どちらも関数オブジェクトの実行を行うもの。
    * 違いは、applyは引数を配列で指定し、callは引数を直接記載する点。
```js
function ghi(arg1, arg2) {
    alert(arg1 + arg2);
}

ghi.apply(null, ["a", "b"]);
ghi.call(null, "x", "y");
```
* 第一引数には、関数オブジェクト内で`this`でアクセスできるオブジェクトを指定する。`null`や`undefined`を指定するとグローバルオブジェクトになる（ブラウザでは`Window`オブジェクト）。


# 変数のスコープ（WIP）
* 関数の中で`var`を使って変数を定義 → ローカル変数
* 関数の中で`var`を使わずに変数を定義 → グローバル変数
* 関数の外で変数を定義 → グローバル変数
* ローカル変数のスコープは関数単位であり、ブロック単位ではない（関数の先頭で宣言するのと同じ扱いになる＝巻き上げ）。
```js
function foo() {
    for (var o in arguments) {
        var x = arguments[o];
    }
    alert(x); // -> "c"
}

foo("a", "b", "c");
```

## スコープオブジェクト
* クロージャの挙動を理解するにはスコープオブジェクトの仕組みの理解が重要。
    * (IMO) クロージャの挙動は、スコープオブジェクトの仕組みとほぼ同義と捉えるとわかりやすい。
* (参考) https://postd.cc/how-do-javascript-closures-work-under-the-hood/
* 例えば以下のようにトップレベルで書いた場合を考える。
```js
var foo = 1;
var bar = 2;
function myfunc() { var a = 1; }
myfunc();
```
* 変数のスコープは、スコープオブジェクトという特別なオブジェクト（連想配列のようなもの）で管理される。
    * `var foo = 1; var bar = 2;` まで実行された時点で、トップレベルのスコープオブジェクト（Global Object）に2つの識別子が定義される（プリミティブなので、値もセットで格納される）。
    * `function myfunc { var a = 1; }` まで実行されると、スコープオブジェクト（この場合はトップレベルなのでGlobal Object）に`myfunc`という識別子が定義され、この識別子とセットで関数オブジェクトへの参照が格納される。
    * 関数オブジェクトは、関数のコードや他のプロパティを持つ。また`myfunc`の関数オブジェクト内には、プロパティとして、関数が定義されたときに有効なスコープオブジェクト（この場合はGlobal Object）への参照が作成される。
    * 関数が呼び出されると、`myfunc`（とその引数の値）のローカル変数を持つ新しいスコープオブジェクトが生成される。この新しいスコープオブジェクトは、関数を呼び出した時に参照したスコープオブジェクト（この場合はGlobal Object）を継承する。
        * `myfunc scope -(継承)-> Global Object`
    * これによって、例えば`myfunc`内から`bar`を参照するとき、まず`myfunc scope`を見に行き、`myfunc`のスコープに`bar`がなければ、継承元のGlobal Objectを見に行くことになる（スコープチェーン）。
* スコープオブジェクトは参照されている限り存続する。逆に、`myfunc()`が完了し`myfunc()`のスコープオブジェクトが参照されなくなると、ガベージコレクションが起動し解放される。
    * つまり、上記の例では呼び出し直後にスコープオブジェクトは解放される。
* `var test = myfunc()` と書いた場合、Global Objectには`test`という識別子と`myfunc`の実行結果への参照が格納されるが、`myfunc`は何も返していないため`test`は`undefined`になる。
* しかし、`myfunc`が以下のようになると話が変わる。
```js
function myfunc() {
    var a = 1;
    return function() { return a++; };
}
```
* この `function() { return a++; }` がクロージャである。
    * 宣言した際、`myfunc`という識別子が定義され、関数オブジェクトへの参照が格納される点までは変わらない。ただし `var test = myfunc()` まで実行されたタイミングで、`myfunc`のスコープオブジェクトの生成に加え、クロージャの関数オブジェクトが生成され、この関数オブジェクトは`myfunc`のスコープオブジェクトへの参照をプロパティとして持つ。`test`にはクロージャの関数オブジェクトへの参照が格納される。
    * さらに`test()`によってクロージャを呼び出すと、クロージャのスコープオブジェクトが定義され、そのスコープオブジェクトは`myfunc`のスコープオブジェクトを継承する。クロージャ自身のスコープオブジェクトは空なので、継承元の`myfunc`のスコープオブジェクトを見に行く。
* まとめると、関数の呼び出し時にスコープオブジェクトが作成される。そのスコープオブジェクトはクロージャによって参照されることでガベージコレクションされない。クロージャを呼び出すたびに更新され、さらに関数を呼び出すたびにスコープオブジェクトは別物として作られるため、これによってクローズドに変数を扱うことができる。
* (IMO) 「クロージャ」という言葉の使い方には注意が必要。単に「スコープオブジェクトで変数を管理する関数」という意味であれば、すべてのJavaScriptの関数がクロージャになってしまう。ここでの「クロージャ」は「実際にスコープオブジェクトを使って、外側のスコープの変数をクローズドに利用している関数」という意味で使っている（ネット上でもこちらの使われ方が多い）。

## ハマりやすい例
* 以下の例は意図通りに動かない。複数のクロージャが同じ`i`を参照してしまうため。
```js
var elems = document.getElementsByClassName("myClass"), i;

for (i = 0; i < elems.length; i++) {
    elems[i].addEventListener("click", function () {
        this.innerHTML = i;
    });
}
```
