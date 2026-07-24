---
title: "モジュール(import/export)"
updated: 2026-07-24
---

[TOP(About this memo))](../README.md) > [一覧(JavaScript)](./README.md) > モジュール(import/export)


# 名前空間（モジュール）
* (参考) https://teratail.com/questions/83675
* ES5以前のJavaScriptには名前空間(namespace)という概念がそもそもなかった。各ファイルは単に結合されるだけだったため、グローバル変数の重複等による問題が発生していた。
* ES6(ES2015)から名前空間の概念が追加された。ただし、ここで追加された名前空間は1つのファイル内で名前空間を分ける手法ではなく、複数のファイルそれぞれが名前空間を持つ方式であり、これがES6から追加されたモジュール(module)である。
* ES6からモジュールとしてスクリプトファイルを読み込む動作ができるようになった。それ以前は、JavaScriptをHTML内で複数羅列した場合、単なるファイルの結合に過ぎなかった。
* モジュール: スクリプトファイル一つ一つが独自の名前空間を持ち、公開された名前を通してしかやり取りできない。

## TypeScriptのnamespace機能
* (IMO) 通常は使わず、JSのモジュール機能を使うべき。大きなプロジェクトでも困ることはないはず。
* (参考) https://maku.blog/p/a3eh9w2/


# export, import
* export / import はES2015 (ES6) の構文。
* exportとは、そのモジュールにあるクラスや変数を外部のファイルからも使えるようにする宣言のこと。
    * 外部のファイル側からはimport文を使うことで利用できる。
    * exportはクラス、関数、オブジェクトなどをエクスポートできる。
* export default
    * 1つのモジュール(jsファイル)には複数の変数・関数・クラスを入れることができ、それぞれに`default`をつけると「このモジュールの中でこのクラスがメインの処理である」ということを表せる。
        * defaultはモジュールにつき1つだけしか宣言できない。
        * importするときに`{}`が不要になる。
            ```js
            import otogizoshi from './export-default.js'
            ```
* curly bracket（波括弧）について
    * `export default App;` のようになっていないものを波括弧を使わずにimportしようとするとエラーになる。
        * (IME) Vue.jsで波括弧をつけずにエラーが出てハマったことがある。
            ```
            import { VueReCaptcha } from 'vue-recaptcha-v3'; でハマった。（{}をつけなかったら以下のエラーが出た）
            Uncaught TypeError: Cannot read properties of undefined (reading 'install')
                at Function.e.use (vue.common.prod.js:6)
                at Module.<anonymous> (app.js:36)
                at n (bootstrap:19)
                at Object.<anonymous> (app.js:1)
                at n (bootstrap:19)
                at bootstrap:83
                at app.js:1
            ```
    * 波括弧をつけることで、特定のプロパティのみimportするようになる。
        * `import React, { useState, useEffect } from 'react';` は、`declare namespace React {` の中の`useState`と`useEffect`だけを持ってくる、という意味になる。
        * (参考) https://qiita.com/taro_kawasaki/items/767dbe84b7eb58111eb3
    * (IMO) 呼び出したいモジュールがReactのようにnamespaceになっている場合は `import React, { 呼び出したいモジュール } from 'react'` のようにし、それ以外でdefault exportになっているものは波括弧をつけず、defaultになっていないものは波括弧をつける、という理解で概ね合っていそう。


# exports / require
* Node自体がサポートしているパッケージの読み込み方（厳密にはCommonJSの仕様）。
* importと比べてどちらを使うかは、プロジェクトやチームの方針による。
    ```
    ・require
        var hoge = require('hoge');
        var hoge = require('hoge').hoge;

        module.exports = hoge;
        exports.hoge = hoge;

    ・import
        import hoge from 'hoge';
        import { hoge } from 'hoge';

        export default hoge;
        export { hoge };
    ```
* exportsとmodule.exportsの違い
    * (参考) https://jovi0608.hatenablog.com/entry/20111226/1324879536
    * (IMO) 迷ったら`module.exports`を使っておけばよい。moduleは現在のモジュールへの参照であり、実際にはグローバルではなく各モジュールごとのローカルとなる。

## 具体例・TIPS
* これだとOK
    * `import AuthUtil from '../../utils/AuthUtil';`
    * `import { AUTH_CLIENTS } from '../../utils/AuthUtil';`
* これだけだと`AUTH_CLIENTS`参照時にエラーになる。
    * `import AuthUtil from '../../utils/AuthUtil';`
* 必要な部分だけimportする場合
    * `import { AUTH_CLIENTS } from '../../utils/AuthUtil';`
