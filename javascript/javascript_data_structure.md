---
title: "データ構造"
updated: 2026-07-24
---

[TOP(About this memo))](../README.md) > [一覧(JavaScript)](./README.md) > データ構造


# 変数宣言（let/const）
* let: 再宣言は不可（再代入は可能）。
* const: 再宣言も再代入も不可。
* let/constはブロックスコープを持つ。
    * 一方varはブロックスコープを持たない。例えばif文の中でvarを使って宣言しても、ブロックの外で参照できる。


# JavaScriptの特徴
* JavaScriptはオブジェクトを中心に成り立っている。関数もオブジェクトである。
* 基本的にJavaScriptでは全てのものがオブジェクトかプリミティブのいずれかとなる。
    * プリミティブ: 数値・文字列・論理値・null・undefined
* 配列もオブジェクトである。ただし、lengthなどの特殊な機能を持っている。
* 関数もオブジェクトである。ただし、関数はかなり特殊なオブジェクトである（詳細は[関数オブジェクト・スコープ](./javascript_function_scope.md)を参照）。


# オブジェクト
* JavaScriptのオブジェクトは基本的にはkeyとvalueのペアを保持するマップ（連想配列）である。
    * `obj['prop'] = value;` あるいは `obj.prop = value;` のようにキーと値のペアを代入すると、オブジェクトが内部的に保持しているマップにキーと値が保存される。
* オブジェクトとMapの違い
    * (参考) https://devsakaso.com/arrays-sets-objects-maps-which-to-use/
    * オブジェクトのキーは文字列となるが、Mapには制約がなく関数などもキーにできる。
    * for-inを使ったループはオブジェクトに対して可能だが、Mapに対しては使えない。
    * for-ofを使ったループはオブジェクトに対しては使えないが、Mapに対しては使える。
* オブジェクトのキーは、クォート（`"`や`'`）を省略可能。
    * (参考) https://jsprimer.net/basic/object/
    * ただし、変数名として利用できないプロパティ名はクォートで囲む必要がある。例えば`my-prop`というプロパティ名は、変数名として利用できない`-`が含まれているため`"my-prop"`とする必要がある。

## オブジェクトリテラル
* オブジェクトへのメンバの登録はオブジェクトリテラルという形式でもできる。
    ```js
    var obj = {
        // var1プロパティを登録
        var1: "hello",
        // funcメソッドを登録
        func: function() {
            alert("world");
        }
    };
    ```


# null, undefined, 0, 空文字(''), false
* `Boolean()`関数は、文字列`"true"`を`true`、`"false"`を`false`に変換してくれるわけではない。文字列が入っていれば`true`、空文字であれば`false`になるだけ。
* Booleanへの変換
    * `!hoge` は、hogeが`null`, `undefined`, `0`, 空文字(`''`), `false`のいずれかであれば`true`になる。
    * `!!hoge` は、型に関わらずbool値を得るためのテクニック。


# 厳密等価
```js
console.log(1 === 1);
// expected output: true

console.log('hello' === 'hello');
// expected output: true

console.log('1' === 1);
// expected output: false

console.log(0 === false);
// expected output: false
```


# 配列
* (参考) https://www.javadrive.jp/javascript/array/index10.html
* 配列もオブジェクトである。ただし、以下のような機能を備えている。
    * lengthプロパティ: 配列のインデックス+1を返す。書き換えると配列のサイズが変わる。
    * pop(): 配列の末尾を削除して、削除した要素を返す。
    * push(value, ...): 配列の末尾に値を追加する。
    * shift(): 配列の先頭を削除して、削除した要素を返す。
    * unshift(value, ...): 配列の先頭に値を追加する。

## 重複削除
* 一度Setに変換してからArrayに戻すと簡潔に書ける。順序の維持やパフォーマンスの面でも扱いやすい。
* (参考) https://qiita.com/netebakari/items/7c1db0b0cea14a3d4419


# Mapオブジェクト
```js
var map = new Map();
map.set('name', '名古屋');
map.get('name');
map.delete('name')

var map2 = new Map([['name', '名古屋'], ['name', '東京']])
```


# 参照渡しと値渡し
* (IMO) JavaScriptには「参照渡し」「値渡し」という言葉だけで単純に説明できる仕組みは存在しないと考えたほうがよい。
    * `b = "hogera"` の場合: 文字列"hogera"を生成してメモリのどこかに置き、bにそのアドレスを格納する。
    * `b[0] = "hogera"` の場合: 文字列"hogera"を生成してメモリのどこかに置き、bに入っているアドレスが指す配列の添字0に対応する部分に、そのアドレスを格納する。
* (参考)
    * https://qiita.com/yuta0801/items/f8690a6e129c594de5fb
    * https://note.affi-sapo-sv.com/js-reference-or-value.php
    * https://qiita.com/mozisan/items/1b9d4bf5a1bb341dd354


# コピーの作成

## ディープコピー
* (参考) https://www.delftstack.com/ja/howto/javascript/javascript-deep-clone-an-object/
* `JSON.parse(JSON.stringify(対象))` で実現できる。
* オブジェクトと配列はこの方法でコピーできる。Mapについては要検証（TODO）。
* 注意
    * ディープコピーは、すべてが別のメモリにコピーされ、参照も新しく作られる。
    * したがって、例えば元のオブジェクトが指していた先と同じものを引き続き使いたい場合は、シャローコピーを使うべきである。

## シャローコピー
* 配列
    * `const newMemberList = Array.from(memberList)` — 配列自体は別のメモリに格納されるが、各要素が指すメモリのアドレスは変わらない（実用上、扱いやすい）。
    * `const newMemberList = memberList` — これはアドレスを格納しているだけなので、コピーとしてはほぼ無意味。
* Map
    * `const copyMap = new Map(copyFromMap)` — copyFromMapの中身をもとに別のメモリにMapを作成するため、中身のキーとvalueは同じアドレスを指す。
* オブジェクト
    ```js
    const copyFrom = { a: "a", b: "b", c: "c" };
    const copy = { ...copyFrom, c: "aaaa" };
    ```
