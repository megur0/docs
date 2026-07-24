---
title: "this・プロトタイプ・class - JavaScript"
updated: 2026-07-24
---

[TOP(About this memo))](../README.md) > [一覧(JavaScript)](./README.md) > this・プロトタイプ・class


# this
* JavaScriptの`this`はJavaやC++の`this`とは全く挙動が異なる。
* JavaScriptの`this`は、ある関数が呼び出された際に、その関数を格納していたオブジェクトを指す。
* (参考) http://www.yunabe.jp/docs/javascript_class_in_google.html
```js
var sayHelloShared = function() {
    console.log("Hello, I'm " + this.name);
};

var alice = {
    sayHello: sayHelloShared,
    name: 'Alice'
};

var bob = {
    sayHello: sayHelloShared,
    name: 'Bob',
    child: alice
};

alice.sayHello();  // Hello, I'm Alice
bob.sayHello();  // Hello, I'm Bob

// 以下の方法も可能
sayHelloShared.call(alice);  // Hello, I'm Alice
sayHelloShared.call(bob);  // Hello, I'm Bob
```
* その他の`this`の挙動については、以下の記事の後半も参考になる。
    * (参考) https://postd.cc/how-do-javascript-closures-work-under-the-hood/


# new
* JavaScriptの`new`もJavaやC++でクラスのインスタンス化を行う`new`とは動きが異なる。
* `new`と一緒に関数を呼び出すと、まず新しい空のオブジェクト（`{}`）が生成される。次に関数が呼び出されるが、その際に関数内の`this`が生成されたオブジェクトを指すようになる。関数が実行された後、生成されたオブジェクトが`new`の実行結果として返される。
```js
var Person = function(name, age) {
    this.name = name;
    this.age = age;
};

var alice = new Person('Alice', 7);

console.log(alice.name);  // Alice
console.log(alice.age);  // 7
```
* 言い換えると、暗黙的に以下を行っていることになる。
    * (参考) https://qiita.com/takeharu/items/809114f943208aaf55b3
```js
var Person = function(name, age) {
    // var this = {}; を実行
    this.name = name;
    this.age = age;
    // return this; を実行
};
```
* ちなみに`Person`に以下のように`return`をつけても意図通り動作する（`return`も上書きされる）。
```js
var Person = function(name, age) {
    this.name = name;
    this.age = age;
    return name;
};
```
* `new`演算子を使うことで、JavaScriptでは関数を「コンストラクタ」として利用できる。実際、`new`で生成されたオブジェクトは`constructor`というプロパティで生成時に利用された関数への参照を保持している。
```js
console.log(alice.constructor == Person);  // true
```
* つまり、`alice`オブジェクトの構造は概念的には以下のようなイメージになる。
    * (?) 正確には`[[Prototype]]: { constructor: Person }`という形になっている。
```js
var alice = {
    constructor: Person,
    name: 'Alice',
    age: 7
}
```


# プロトタイプ
* prototypeには2種類ある。
    * (参考) https://tatsuno-system.co.jp/2020/03/16/blog_java-script/
    * 関数オブジェクトが持つ`prototype`
        * 直接参照することができる。
        * 中身はオブジェクトであり、`constructor`プロパティのみを持つ。このプロパティは関数自身を参照する。
        * (?) 継承を行った場合はその親を参照するようになる。
    * オブジェクトが持つ内部プロパティ`[[Prototype]]`
        * 直接参照することはできない。参照するには`Object.getPrototypeOf()`を使う。
        * (?) これも中身はオブジェクトになっていて`constructor`プロパティを持ち、そのプロパティは`new`した際にひな形になった関数オブジェクトへの参照になっている。
        * 便宜上、オブジェクトが持つ`[[Prototype]]`を「オブジェクト.[[Prototype]]」と表記する。
* プロトタイプチェーンとは
    * オブジェクトのプロパティを参照しようとしたときに、そのプロパティがなければ「オブジェクト.[[Prototype]]」にプロパティを探しに行く仕組みのこと。
* 使い方: 関数でプロトタイプ（雛形）を作って`new`する。
```js
function Animal() {
    this.cry = "meow";
}
Animal.prototype.shout = function() { // 関数が持つプロトタイプは直接参照できる
    alert(this.cry);
}
var animal = new Animal();
animal.shout();  // -> "meow"

function Bird() {
    this.cry = "sing";
}
Bird.prototype = Object.create(Animal.prototype); // 継承

var niwatori = new Bird();
niwatori.shout();  // -> "sing"
```
* `niwatori`オブジェクトは概念的には以下のようなイメージになる。
```js
var niwatori = {
    [[Prototype]]: {
        constructor: Birdへの参照
    },
    cry: "sing"
}
```
* このとき、以下のような動作をプロトタイプチェーンという。
    * `niwatori.shout()` → `niwatori`のメンバには`shout`が存在しない
    * → `niwatori`を生成した`Bird`を見に行く（`niwatori.[[Prototype]]`の`Bird`への参照から見に行く）
    * → `Bird.prototype`を生成した`Animal.prototype`を見に行く（`Bird.prototype.constructor`を見に行く）
    * → `Animal.prototype`を見ると`shout`を持っている
* メリット
    * これによってJavaなどのクラスのように処理を再利用できる。
    * 例えば以下のように書いてしまうと、一見同じ処理をしているように見えるが、挙動が異なる。
```js
function Animal() {
    this.cry = "meow";
    this.shout = function() {
        alert(this.cry);
    };
}
var animal = new Animal();
animal.shout();  // -> "meow"
```
    * この書き方の場合、暗黙的に以下の実行になっているため、`new`のたびに新しい関数オブジェクトが生成されてしまう（プロトタイプの場合は実行時にプロトタイプチェーンで探しにいくため再利用できる）。
```js
function Animal() {
    // var this = {}; を実行
    this.cry = "meow";
    this.shout = function() {
        alert(this.cry);
    };
    // return this; を実行
}
```


# ミックスイン
* ミックスインの説明の図を見ると、prototypeやsuperの理解にも役立つ。
* (参考) https://ja.javascript.info/mixins


# class
* Class構文はprototypeベースのクラス定義構文の糖衣構文。
* 従来のprototypeベースの構文より、簡潔かつ明瞭にクラスを定義できる。
* ES2015から導入された。
* (参考) https://qiita.com/soarflat/items/b251caf9cb59b72beb9b

## prototypeベースの構文
```js
var Person = function(name) {
    this.name = name;
};
Person.prototype.sayHello = function() {
    console.log("Hello, I'm " + this.getName());
};
Person.prototype.getName = function() {
    return this.name;
};
```

## Class構文
```js
class Person {
    constructor(name) {
        this.name = name;
    }
    sayHello() {
        console.log("Hello, I'm " + this.getName());
    }
    getName() {
        return this.name;
    }
}
```

## 継承
```js
class Person {
    constructor(name) {
        this.name = name;
    }
    sayHello() {
        console.log("Hello, I'm " + this.getName());
    }
    getName() {
        return this.name;
    }
}

// Personを継承
// 同じメソッド名であるsayHelloは継承先のもので上書きされる
class Teacher extends Person {
    sayHello() {
        console.log("Hi, I'm " + this.getName());
    }
}

const teacher = new Teacher('soarflat');
teacher.sayHello(); // => Hi, I'm soarflat
```


# コールバック関数内でthisを参照するとき
* コールバック関数の中で外側の`this`を参照したい場合、以下のように`let re_this = this`のような形で退避しないと参照できないことがある。`this.xxxx = token`のように直接書くと`undefined`になる。
```js
data() {
    return {
        token: ""
    }
},
mounted() {
    let re_this = this
    grecaptcha.ready(function() {
        grecaptcha.execute('サイトキー').then(function(token) {
            re_this.token = token
        });
    });
}
```
* 上記の`this`はオブジェクト型（プリミティブではない）であるため`re_this`には参照が渡される。その結果、`re_this.token = token`によって参照先を更新できる。
* コールバック関数内の`this`は`window`オブジェクトになってしまうことがある。
    * アロー関数だと（親コンテキストに束縛できないため）参照できず、`function`の短縮記法であれば参照できる。さらにコールバック関数側をアロー関数にすると参照できる、という分かりづらい仕様になっている。
    * (参考) https://tadaken3.hatenablog.jp/entry/vue-scope-this


# jQueryの$
* JavaScript自体としては、`$`は単なる変数名として使える記号。
* jQueryでは`$`を関数として使っている。jQueryの`$`はユーティリティ的ないくつかのプロパティやメソッドを持つ。
* `$`を関数として呼び出す（`$("セレクタ")`など）とjQueryオブジェクトを返す。
* jQueryオブジェクトのコンストラクタの`prototype`プロパティは`$.fn`である。つまりjQueryオブジェクトは`$.fn`を継承しているため、`$.fn.Hoge`メソッドを作れば、すべてのjQueryオブジェクトに`Hoge`メソッドができる。
    * プラグインはこのようにして`$.fn`を拡張することでjQueryオブジェクトを拡張できる。
* (参考) https://www.task-notes.com/entry/20140713/1405246477
* 例
```js
// bullyを作ってしまえば、全てのjQueryオブジェクトについてbullyが使える
$.fn.bully = function() { /* ... */ }
$('#' + id).bully();
```
* セレクタの指定例
    * 初期化の際にcheckedされているradioの値を取得する例
    ```js
    let sc_sel_val = $('label.sc_radio_label > input:radio:checked').val();
    ```
