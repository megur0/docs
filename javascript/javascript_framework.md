---
title: "仮想DOM・フレームワーク比較 - JavaScript"
updated: 2026-07-24
---

[TOP(About this memo))](../README.md) > [一覧(JavaScript)](./README.md) > 仮想DOM・フレームワーク比較


## 仮想DOM
* クライアントサイドレンダリングでは、JavaScriptで多くの画面要素を描画することになる。ここで避けて通れないのがDOM（Document Object Model）である。
* DOMは、HTMLの各タグ（要素）をノードとみなし、入れ子構造を階層構造で表現するモデルに変換することで、HTMLをJavaScript等で扱えるようにした技術。
* 素のJavaScriptやjQueryなどでDOM操作を直接行う場合、プログラマがDOM構造を意識する必要があり、コードが長くなりメンテナンス性が悪化しがちである。
* これを解決するための「仮想DOM」という技術を取り入れたJavaScriptフレームワークが現在の主流であり、代表的なものとして「Angular」「React」「Vue.js」などがある。
* 仮想DOMは、実際のDOMを構築する前にバーチャルなDOMを構築するワンクッションを入れることで、プログラマが実DOMを直接意識しなくて済むようにする。
* さらに、バーチャルなDOMを構築する際に差分更新する仕組みを取り入れることで、画面更新を高速化できる。
* (参考) https://arakan-pgm-ai.hatenablog.com/entry/2019/04/18/000000
* (参考: 仮想DOMの詳しい話) https://eh-career.com/engineerhub/entry/2020/02/18/103000

### クリティカル・レンダリング・パス
* (参考) https://entry.anypicks.jp/improve-html-and-css-performance/
* DOMとCSSOMを作ってからレンダーツリーを構築する。
* DOMはHTML5以降、ストリーミング（読み込みながら逐次レンダリング）できるようになった。
* その後、画面サイズに合わせたレイアウト決定と描画が行われる（これは画面サイズを変えるたびに再実行される）。


## フレームワーク比較
* (参考: 2018年の記事) http://iwasiman.hatenablog.com/entry/2018/04/23/200000
* (IMO)(?) 以下は上記の2018年時点の記事をもとにした内容であり、フレームワークの動向はその後も変化し続けている（React Hooks、Vue 3 Composition API、Svelte/SolidJSの登場など）。個別の技術詳細よりも、当時のSPAフレームワークに関する考え方の一例として捉えるのがよさそう。
* ReactやAngularはDOMをラップ（抽象化）してくれるため、DOMを意識せずアプリのコンポーネントだけを考えれば実装できる（DOMは注意しないとブラウザ差異やパフォーマンス低下が起きることがある）。
* SPAは結局、なるべく状態を持たないシンプルな設計にすることがポイントとなる。
* 動作速度は両者それほど変わらない。
* minifyについて: 当時はAngularの方がコードが細分化されており、コンパイル後にファイルサイズを小さくしやすいとされていた。
* 基本的にAngularもReactも、jQueryなどDOM操作を行うプラグインとの共存は苦手。
* DOMに転写するオーバーヘッドがあるため、ハイパフォーマンスが求められるゲームには不向き。
* ブログやWebメディアなど、静的なテキスト情報を掲載するサイトにも不向き。基本的にクローラーはJavaScriptを実行できないためSEO的に不利になりやすい（今後変わる可能性はある）。サーバーサイドレンダリングをすれば静的にできるが、一手間かかる。
* Reactは他のライブラリと組み合わせて使うことが前提のため、比較的上級者向けとされる。コードはJavaScript。
* Angularは基本的に単体で使う（Angularを使えば安心して全体を作れるという思想のため）。コードはTypeScript。またRxJSというライブラリに依存しているため、その理解も必要になる。非同期処理（XHR、Promise）に慣れる必要もある。
* (参考) https://www.google.co.jp/amp/s/employment.en-japan.com/engineerhub/entry/2018/04/13/110000%3famp=1
