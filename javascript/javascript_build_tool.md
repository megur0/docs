---
title: "Babel・webpack - JavaScript"
updated: 2026-07-24
---

[TOP(About this memo))](../README.md) > [一覧(JavaScript)](./README.md) > Babel・webpack


# Babel
* Transpilerとは
    * Code to Codeの変換を行うツール。
    * ECMAScriptにおいてはES6以降のコードをES5のコードに変換するBabelがある。
    * CSSではPostCSSなどが同様の役割を持つ。
* (IMO) ずっとBabelを使い続けることになるのか？
    * 仕様策定の観点では、多く使ってもらえるとフィードバックが得やすいため歓迎されている。
    * 普通に使うだけであればES6の機能とライブラリで十分なケースも多い。
    * 新しい仕様を積極的に試したいわけでなければ、Babelを使わないという選択肢もありえる。


# webpack
* モジュールを結合するツール。importで書かれた依存関係を解決して結合してくれる。
* importが使えないブラウザでは必須。importが使えるブラウザであっても、webpackを使わないとモジュールの数だけ通信が発生してしまうため、使った方が良い。
* インストール
    ```
    npm install --save-dev webpack webpack-cli webpack-dev-server
    ```
    * 上記の3つがインストールされる（`--save-dev`のため`devDependencies`に記載される）。


# babel（ビルドでの利用）
* JSのコードを解析し、古いランタイムでも動作するように変換する。
* ES6のコードをES5に自動的に変換できる（トランスパイル）。
    ```
    npm install --save-dev babel-loader @babel/core @babel/preset-env
    ```

## webpackでビルドする手順
* (参考) https://qiita.com/civic/items/82c0184bcadc50965f91

## bootstrapをwebpackで使う
* (参考) https://ics.media/entry/17749/
    ```
    npm i -D style-loader css-loader
    npm i bootstrap jquery popper.js
    npm i bootstrap-confirmation2
    ```
* 上記のようにパッケージをインストールする（node_modulesにインストールされ、package.jsonのdependenciesに追加される）。

## npm、webpack、babelを組み合わせた構成例
* (参考) https://qiita.com/riversun/items/29d5264480dd06c7b9fb
* npmでパッケージ管理、webpackでJSをまとめる、babelでトランスパイルする、という組み合わせ。
* 構成例
    ```
    ├── dist
    │   └── app.js
    ├── node_modules
    │   └── ...
    ├── public
    │   └── index.html
    ├── src
    │   ├── hello.js
    │   └── index.js
    ├── .gitignore
    ├── package.json
    ├── package-lock.json
    └── webpack.config.js
    ```
