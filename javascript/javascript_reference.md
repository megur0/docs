---
title: "参考リンク・コミュニティ・仕様 - JavaScript"
updated: 2026-07-24
---

[TOP(About this memo))](../README.md) > [一覧(JavaScript)](./README.md) > 参考リンク・コミュニティ・仕様


## 学習に使えそうなもの
* JavaScript Primer
    * https://jsprimer.net/
    * オブジェクトなどの説明が学習の参考になる（例: https://jsprimer.net/basic/object/ ）。

## 参考
* Google流JavaScriptにおけるクラス定義（IME: 学習途中の内容）
    * http://www.yunabe.jp/docs/javascript_class_in_google.html
* JavaScriptオブジェクトの基礎
    * https://qiita.com/yoshi389111/items/245df2d642e49d2acf3a
    * ES5の話なので、情報が古い可能性がある。ES6以降で変わっている点があるかもしれない。


## コミュニティ・団体

### Mozilla
* Mozilla Foundation
    * ネットスケープ（後述）が次世代のインターネットスイートを開発するために設立したフリーソフトウェア・オープンソースプロジェクトの名称に由来する。1998年に設立され、2003年に非営利団体化した。
    * 現在はMozilla FirefoxやMozilla Thunderbirdなどの開発・保守を行っている。
* HTMLレンダリングエンジンGeckoを搭載したブラウザを指す場合もある（Mozilla Suite、Mozilla Firefox、Camino、Netscape Navigator 9など）。

### ネットスケープコミュニケーションズ
* ジム・クラークとマーク・アンドリーセンらによって設立された、かつて存在したアメリカの企業。
* 1994年にMosaic Communications Corporationとして設立され、同年にネットスケープコミュニケーションズへ社名変更。1998年にAOLに買収された。
* Netscape Navigator（ウェブブラウザ）に加えて、JavaScript、RDF/RSS、SSLといった技術を生み出した。
* Netscape Navigatorの無料配布戦略によりWorld Wide Webの利用が拡大し、これに対抗する形でマイクロソフトがInternet Explorerをリリースした（第一次ブラウザ戦争）。
* Netscape製品は、後にAOLが設立したMozilla Foundationに引き継がれ、開発が続けられている。

### MDN Web Docs
* ウェブ標準およびMozillaプロジェクトの開発文書のための公式サイト。
* 旧称はMozilla Developer Network、さらに以前はMozilla Developer Center。


## 仕様・標準化団体

### ECMAScriptとは
* (参考) https://azu.github.io/slide-what-is-ecmascript/

### TC39
* Technical Committee（専門委員会）の略。
* TC39はECMAScriptを策定している専門委員会。
* Ecmaは様々な仕様を策定しており、その中でECMAScriptを策定しているグループがTC39。
    * ちなみに、同じくEcma標準化されているDartはTC52。

### ECMA-262
* ECMAScriptの正式な仕様番号。262はEcma Internationalでの管理番号。
* 仕様: https://github.com/tc39/ecma262
    * 多くの仕様がGitHub上で管理されている（W3CやWHATWGにも同様の傾向がある）。
* Web版: https://tc39.es/ecma262/
    * (?) Ecmarkupという仕様書向けのカスタム要素を定義したHTMLで書かれている。
* 仕様に関する議論は上記リポジトリのissueで行われている。

### エディタ（仕様策定者）
* ES6の仕様策定のリーダーはAllen Wirfs-Brock（アレン・ワーフスブラック）。

### Ecma標準とISO標準の違い
* ECMAScriptはデファクト標準（Ecma Internationalにより標準化）であると同時に、ISO/IEC 16262としてISO標準化（デジュール標準）もされている。
* 日本ではISO/IEC JTC1/SC22のECMAScript adhoc委員会でFast Trackの手続きが行われている。

### 仕様の策定プロセス
* ECMAScript 2016以降は、機能ごとに仕様のプロポーザル（提案）を出して策定される。
* 各プロポーザルには「Stage」と呼ばれる5段階のラベルがある。
* 1年ごとに新しいECMAScript仕様がリリースされる（ES6はリリースまで結局6年かかったため、それ以降はスピードアップする方針になった）。
* Stage 4となったプロポーザルは次期ECMAScriptに取り込まれ、正式な仕様となる。
    * 0 Strawman - アイデア
    * 1 Proposal - 提案
    * 2 Draft - ドラフト
    * 3 Candidate - 仕様書と同じ形式
    * 4 Finished - 策定完了
* (IMO) ブラウザに実装されることと仕様に入ることは別物である点に注意。ブラウザで実装されても、必ずしも仕様に入るとは限らない。

### TC39 proposal
* プロポーザルはブラウザベンダー、ウェブ開発者、個人の誰でも行うことができる。
* 各プロポーザルのステージなどは以下に記載されている。
    * https://github.com/tc39/proposals
* プロポーザルの大まかな流れ
    * tc39/proposalsのstage0.mdにプロポーザルを追加してPull Requestを送る。
    * 仕様策定のプロセスを理解する。
    * フォームから必要な情報を送りルールに同意する。
    * ProposalをPull Requestする。


## ES6以降の変遷

### ES6（ES2015）
* ECMAScriptの6th Editionのこと。6th editionの「6」からES6と呼ばれていたが、2015年に標準化されたため正式名称はES2015となった。
* リリースまでに6年かかったため、変更内容のボリュームが大きい。
* ブラウザ対応状況: https://kangax.github.io/compat-table/es6/
    * (?) 執筆時点ではIE11が非対応だったが、IE11自体が2022年にサポート終了しているため、現在は主要ブラウザでほぼ問題なく利用できると考えられる。
* 主な新機能
    * let/const、ブロックスコープ
    * アロー関数
    * クラス、継承
    * 関数の引数のデフォルト値
    * 分割代入
    * テンプレート文字列（``で囲むと変数展開ができる。改行も反映可能）
    * スプレッド構文（可変長引数（レストパラメータ）、配列の展開、分割代入時の値のまとめ、配列の結合）
    * for...of、Promise
* (参考) https://qiita.com/rana_kualu/items/1f98c1a642102f48aa78

### ES2016〜ES2020
* ES2016: 軽微な変更のみ。
* ES2017: async/awaitが追加された。
* ES2018: Asynchronous iterationが追加された。Node.jsのStreamは扱いが難しいと言われることがあるが、これをシンプルに扱えるようになった点が特徴。
* ES2019
    * (参考) https://qiita.com/tonkotsuboy_com/items/07f8ef98abf89250b90c
    * (?) 執筆当時のブラウザ対応状況についての言及があったが、現在は主要ブラウザでほぼ対応済みと考えられる。
* ES2020: dynamic import（動的読み込み）が追加された。
    * これまでのimportはJavaScriptのトップレベルでしか用いることができず、モジュールは即座に読み込まれる仕様だった。
    * dynamic importでは `import(モジュール名)` を用いてモジュールを動的に読み込む。返り値はPromise。
        ```js
        // import()のタイミングでsub.jsが読み込まれる
        import('./sub.js')
            .then(module => {
                // 動的に読み込まれたSubクラス
                const sub = new module.Sub();
                sub.subMethod();
            });
        ```
    * 従来の書き方（この場合は即座にsub.jsが読み込まれる）
        ```js
        import { Sub } from './sub.js'

        const sub = new Sub();
        sub.subMethod();
        ```
    * (参考) https://qiita.com/ryoya41/items/a80efb6fbb9edc4861d4
    * Safariは11.1、Firefoxは67、Edgeは79からdynamic importに対応している。iOS Safariでも動作する。
        * (参考: Can I use) https://caniuse.com/es6-module-dynamic-import

### ES2021
* `Promise.any()` / `AggregateError`（複数のPromiseのうち、いずれか1つでもresolveされたらresolveする。全て失敗した場合は`AggregateError`でreject）
* `String.prototype.replaceAll()`（正規表現のgフラグを使わずに、文字列の全ての一致箇所を置換できる）
* `WeakRef` / `FinalizationRegistry`（オブジェクトへの弱参照や、ガベージコレクト時のコールバック登録）
* 論理代入演算子（`&&=`, `||=`, `??=`）
* 数値セパレータ（`1_000_000`のように`_`で桁を区切って記述できる）

### ES2022
* トップレベルawait（モジュールのトップレベルで、`async`関数で囲まずに`await`が使えるようになった。詳細は[非同期処理・イベントループ](./javascript_async.md)を参照）
* クラスのprivateフィールド・privateメソッド（`#field`のような記法でクラス外からアクセスできないメンバを定義できる）
* クラスのstaticブロック（クラス定義内で`static { ... }`という初期化ブロックを書ける）
* `Error.prototype.cause`（エラーオブジェクトに、原因となった別のエラー等を保持できる）
* `Array`/`String`/型付き配列の`at()`メソッド（負のインデックスで末尾から要素を取得できる）
* `Object.hasOwn()`（`Object.prototype.hasOwnProperty.call(obj, key)`の簡潔な代替）

### ES2023
* `Array.prototype.findLast()` / `findLastIndex()`（配列の末尾から要素を検索する）
* 配列を破壊的に変更せず新しい配列を返すメソッド群: `toReversed()` / `toSorted()` / `toSpliced()` / `with()`
* Hashbang構文（`#!`で始まるスクリプトファイルがサポートされ、Node.jsなどのシェバン行付きスクリプトをそのままパースできるようになった）

### ES2024
* `Object.groupBy()` / `Map.groupBy()`（配列の要素を、コールバックの返り値に基づいてグルーピングする）
* `Promise.withResolvers()`（`promise`、`resolve`、`reject`をまとめて返すユーティリティ）
* `ArrayBuffer` / `SharedArrayBuffer`のリサイズ・転送機能
* 正規表現の`v`フラグ（Unicodeの集合演算などに対応した拡張フラグ）
* `String.prototype.isWellFormed()` / `toWellFormed()`（文字列が正しい形式のUnicodeかどうかの判定・修正）

### ES2025
* Iterator helpers（新しい`Iterator`グローバルオブジェクトが追加され、イテレータに対して`map()`/`filter()`/`take()`/`drop()`等のメソッドが直接使えるようになった）
* Setの集合演算メソッド（`union()`、`intersection()`、`difference()`、`isSubsetOf()`等）
* 正規表現の名前付きキャプチャグループが、別の分岐（alternative）間であれば重複して使えるようになった（従来は構文エラーだった）
* (参考) https://pawelgrzybek.com/whats-new-in-ecmascript-2025/
* (?) 対応状況の一例（執筆時点）: Chrome 122+、Node 22+、Firefox 131+、Safari 18.4+。今後変わる可能性がある。

### 現在進行中・注目のプロポーザル（IMO・執筆時点の情報）
* Temporal（日付・時刻を扱う新しいAPI）
    * 長年Stage 3に留まっていたが、2026年3月のTC39会議でStage 4に到達し、ES2026の一部となった。
    * Firefox（139+）、Chrome（144+）、Edge（144+）ではすでにネイティブでサポートされている。Safariや多くのモバイルブラウザは執筆時点では未対応とされる。
    * Node.jsでは、Node 26（2026年5月リリース）からフラグなしで利用可能になったとされる。
    * 従来の`Date`オブジェクトが抱えていたタイムゾーンの扱いにくさ等の問題を解消することを目的としている。
* Decorators（デコレータ構文）
    * クラスやクラスメンバーに`@デコレータ名`という記法でメタプログラミング的な処理を追加できる構文の提案。
    * (?) 仕様としてはある程度まとまっているが、主要ブラウザ間でどこが最初に実装・出荷するかを様子見している状況が続いており、執筆時点でも本格的な普及には至っていない模様。
* Records & Tuples
    * オブジェクトや配列のイミュータブル（不変）版を言語に追加する提案だったが、パフォーマンス面の懸念などから合意形成が難航し、2025年4月のTC39会議でStage 2の段階のまま提案自体が取り下げられた。


## CommonJS
* サーバーサイドなどウェブブラウザ環境外におけるJavaScriptの各種仕様を定めることを目標としたプロジェクト。
* ECMAScriptがブラウザ上でのJavaScriptの仕様と標準を作っているのに対し、CommonJSはブラウザ上に限らず、サーバーサイドやクライアントのCUI/GUIでJavaScriptを使う際の仕様を作成している。
* CommonJSの仕様を実装しているソフトウェアの例としてNode.jsがある。
