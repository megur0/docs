---
title: "非同期処理・イベントループ - JavaScript"
updated: 2026-07-24
---

[TOP(About this memo))](../README.md) > [一覧(JavaScript)](./README.md) > 非同期処理・イベントループ


## データ取得（fetch）
* fetchを使う。
    * 従来、このような機能はXMLHttpRequestを使用して実現されてきた。
    * fetchはそのより良い代替となるもので、Service Workerのような他の技術からも簡単に利用できる。
* fetchは、CORSやHTTPへの拡張のような、HTTPに関連する概念をまとめて定義している。
* (参考) https://developer.mozilla.org/ja/docs/Web/API/Fetch_API/Using_Fetch


## 非同期処理・イベントループについて思うこと（IMO）
* (IMO) JavaScriptの非同期処理は、歴史的に大まかに以下のように発展してきたと理解している。
    * もともとJavaScriptは、非同期のAPIと処理を同期させるためにコールバック関数を渡す形式だった。ただし、依存関係のある非同期API呼び出しが増えるとコールバックが多重にネストし、可読性が著しく低下するという問題があった（いわゆるコールバック地獄）。
    * そこで`Promise`、`then`、`Promise.all`といった仕組みが登場した。非同期APIを呼ぶ関数側でPromiseとしてラップして返し、呼び出す側は`then`や`Promise.all`によって同期的に扱えるようにした。
    * ただし、Promiseの書き方には癖があるため、その後`async`/`await`が導入された。
* (IMO) `async`/`await`から学び始めると誤解しやすい点があると感じている。`async`は関数につけることでその関数がPromiseを返すようにラップしてくれるが、非同期処理の実体（setTimeoutやfetch等の外部APIが時間のかかる処理を行う部分）自体は`async`をつけたことによって生まれるわけではない。非同期なAPIを同期的に扱うための手段が、コールバック→Promise→async/awaitと文法を変えてきただけで、目的自体は変わっていない。
    * (?) ただし、外部APIを一切呼ばなくても、Promiseや`await`自体がマイクロタスク経由で処理を1回分遅延させる（`await null;`のような場合でも、その後の行は現在の同期処理が終わってから実行される）。したがって「非同期なのは呼び出した外部APIだけで、Promise/async自体は何も非同期的な性質を持たない」と言い切るのはやや言い過ぎで、Promiseの仕組み自体（マイクロタスクによるスケジューリング）もJavaScript（ECMAScript）の言語仕様の一部として定義されている点には注意が必要。
    * (IMO) ネット上の記事で「JSで非同期処理をする」という表現が増えることで、「JSは非同期処理ができる（JS自体が並行に処理できる）」という誤解が広がりやすいように感じる。
    * (IMO) JSがシングルスレッドであり、コールスタック上の処理はブロッキングであること、その上でブラウザや外部のAPIが非同期に動き、それをコールバックやPromiseで同期させているという構造を理解しないと、この誤解を解消しづらい。
    * イベントループの仕組みを理解しないと、ブラウザ上で動作するJavaScriptコードの挙動を正しく説明できない（例: setTimeoutの挙動）。
* (IMO) 初学者にとっては、このあたりの仕組み（コールスタック・イベントループ・WebAPIs・タスクキュー等）を理解しないまま非同期処理のコードだけを覚えると、遠回りしやすい分野だと感じる。

### (IMO) awaitがasyncの中でしか使えない制約について
* この制約の理由は、async/awaitの内部実装に起因すると理解している。
* (IMO) この制約は、awaitの用途から考えた本質的な必然性というより、「JavaScript」という言語の実装上の都合による制約という側面が大きいのではないかと感じている。
    * (?) 一方で、これは単なる実装都合とも言い切れない面がある。通常の関数は呼び出されたら必ず一度で実行が完了する（実行の途中で他の処理に制御を明け渡さない）というJavaScriptの前提があり、`await`はその前提を崩して関数の実行を中断・再開できるようにする仕組みである。`async`という明示的なマーカーを必須にすることで「この関数は中断されうる」ということが静的に分かるようにしている、という見方もでき、その意味では一定の設計上の合理性があるとも考えられる。
    * (?) awaitという仕組み自体が、内部実装として非同期処理のような扱いをしないと成立しないものなのかもしれない
        * 一方で、`await`をしている関数が普通の関数（`async`なし）だった場合、外部から見てその関数が内部で時間のかかる処理をしているかどうかが分かりにくくなるため、`async`が明示されていた方が読みやすいという見方もできる。なお、`main`関数に関しては`async`である必要がない場合もある。
* (参考)
    * https://qiita.com/jun1s/items/ecf6965398e00b246249
    * https://qiita.com/jun1s/items/4ec151b1c2562f6b0f26
* (IMO) 上記の参考記事では「awaitの処理は内部的に非同期処理になっているからasyncの中でしか使えない」「Promiseの糖衣構文だから」という説明がされている。とはいってもasync/awaitという命名から挙動を初学者は推測しづらい（学習コストの高い）とは感じる
* (参考: Goのgoroutineとの対比についての記事) https://zenn.dev/nobonobo/articles/9a9f12b27bfde9


## 参考記事

### イベントループ
* https://qiita.com/guanghuihui/items/57dcc7cb867eee951f36
* https://meetup-jp.toast.com/896

### 仕様
* HTMLスペック（WebAPIs）: https://html.spec.whatwg.org/multipage/webappapis.html#hostenqueuepromisejob
* ECMAScript: https://tc39.es/ecma262/#sec-intro
* Promiseの仕様: https://promisesaplus.com/
* MDN: https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/await
    * Mozillaの公式サイト。一次ソースではないが、ドキュメントはしっかりしている。

## ブラウザで動くJSは分かりづらい
* プログラマーがコールスタックやイベントループ、WebAPIs、タスクキューを含めた動きを理解していないと、コードの挙動が理解できないことがある。
    * 例えば、try〜catchをなぜコールバック内に渡す必要があるのかは、スタックの状態が分かっていないと納得しにくい。
    * (参考) https://meetup-jp.toast.com/896
* (IMO) JSの仕様（ECMAScript）自体よりも、ブラウザ上でJSを扱う部分（イベントループ等）の理解の方が難しいと感じる。


## JavaScript本体と、JavaScriptが駆動する環境は別物
* JavaScript本体（JavaScriptランタイム・エンジン）は、シングルスレッドで動作する。
    * あくまでもJavaScriptエンジン自体がシングルスレッドで動いているのであって、実際にはブラウザ側でJavaScript以外の機能（後述）も動いているため、全体としてはマルチスレッドで動いている（この区別が重要）。
    * JavaScript本体はシングルスレッドで同期的に動く（単一のコールスタックで処理する）言語だが、JavaScriptエンジンを駆動する環境（ブラウザやNode.js）はマルチスレッドかつ非同期に動作する。この二重構造がJavaScriptを分かりづらくしている一因と考えられる。
    * JavaScriptが「ノンブロッキングな実行が可能」と言われるのは、主にこの外側の環境（WebAPIsやNode.jsのI/O)が非同期に動くことを指している。
    * (?) ただしECMAScript自体が非同期処理に一切触れていないわけではない。Promiseや、その基盤となる抽象的な「Job Queue」（非同期に実行されるジョブをキューへ積む仕組み）はECMAScript仕様の一部として定義されている。ECMAScriptが定義していないのは、そのJobを具体的にいつ実行するかというスケジューリングの詳細（タスクとマイクロタスクの区別や、レンダリングとの兼ね合いなど）であり、それを定めているのがHTML仕様（ブラウザの場合）である。
        * イベントループの具体的な処理手順はHTMLの仕様に記載されている: https://html.spec.whatwg.org/multipage/webappapis.html#event-loops
* コールスタック、イベントループ、コールバックキューやその他のAPIをブラウザ側が保持している。

### ランタイム
* コールスタックとヒープメモリを持つ。
    * ヒープメモリはメモリのアロケーション、コールスタックはスタックフレームなどの管理を担う。
* setTimeoutやDOM、HTTPリクエスト（XMLHttpRequest）はJSランタイムには含まれない。これらはブラウザが提供する機能（WebAPIs）である。
* V8
    * Google Chromeで使われているオープンソースのJavaScriptエンジン。C++で書かれている。
    * https://github.com/v8/v8

### コールスタック
* 関数のスタック。呼び出すとpush、returnでpopされる。
* スタックオーバーフロー
    * 例えば、関数が自分自身を無限に呼び出せばオーバーフローを起こすことができる。

### イベントループ、タスクキュー、WebAPIs
* JSエンジンはシングルスレッドで、コールスタックは1つだけしか持たない。つまりランタイムは一度に1つのことしかできない。
* しかし、実際には非同期に動くものが存在する。setTimeoutやXMLHttpRequestのようなI/O待ちが発生するものについては、ブラウザがランタイム（JavaScriptエンジン）以外の機能も持っているためである（この意味では、非同期なのはJavaScript自体ではなく、JavaScriptが駆動するブラウザ側と言える）。
    * (?) ただしPromise/async・awaitによるマイクロタスクの遅延自体は、外部のI/Oを介さなくてもJavaScript（ECMAScript）の言語仕様自体の中で発生する点には注意（詳細は「awaitがasyncの中でしか使えない制約について」を参照）。
    * これらの機能が、WebAPIs（setTimeout、addEventListener、XMLHttpRequest等）、イベントループ、タスクキュー等である。
* コールスタック、イベントループ、タスクキュー、WebAPIsによって非同期処理が実現される。
* WebAPIsが提供するAPIがコールスタックに入ると、コールスタックはWebAPIsに対してそのAPIを実行するように伝える（その際に引数とコールバック関数も渡す）。
* イベントループはコールスタックとタスクキューを監視する。コールスタックが空になると、タスクキューからコールスタックへpushする。
* (参考: 動きを可視化できるツール) http://latentflip.com/loupe/

### Node.js
* Node.jsもブラウザと同様に非同期IOを実現しているが、その実装であるlibuvライブラリが提供するイベントループは、ブラウザのタスク/マイクロタスクモデルとは異なる構造（タイマー、I/O、close callback等のフェーズに分かれた構造）を持つ。
* (?) Node.jsはブラウザではないためHTML仕様に準拠する対象ではなく、「HTML仕様に準拠しきれていない」というより「ブラウザとは別の設計のイベントループを持つ」と捉えるのが正確。そのため、ブラウザ環境のイベントループとは詳細な実装が異なる（そもそもブラウザ同士でも細部は異なる）。
* (参考) https://meetup-jp.toast.com/896

### イベントループの具体例
* WebAPIsとtry〜catchとコールスタック
    * WebAPIsを呼び出す際、try〜catchで囲んでもキャッチできない。コールバック関数の中にtry〜catchを入れる必要がある。
    * 理由: コールバック関数はWebAPIsの処理が完了した後にタスクキューに入れられ、コールスタックが空になったタイミングでイベントループによってタスクキューからコールスタックへpushされる。その時点では外側の文脈（try〜catchを書いたスタックフレーム）はすでにスタックから外れているため、外側にtry〜catchを書いてもキャッチできない。
    * (参考) https://meetup-jp.toast.com/896
* setTimeout
    ```js
    setTimeout(function() {
        console.log('A');
    }, 0);
    console.log('B');
    ```
    * 上記はB→Aの順番で実行される。まず`setTimeout`の呼び出しがスタックに積まれ、WebAPIsの`setTimeout`が呼ばれる。スタック側は`setTimeout`の呼び出しが完了次第すぐにpopされ、次に`console.log('B')`がpushされて実行される。WebAPIs側では、指定時間経過後にコールバック関数がタスクキューに入る。そしてイベントループによって、スタックが空になった後（つまり`console.log('B')`が完了した後）にタスクキューからスタックへ移されコールバック関数が実行される。したがってB→Aの順番になる。
    * この性質上、`setTimeout`はコールスタックとマイクロタスクキュー（後述）がすべて空になってから実行される。つまり、指定した秒数ぴったりで実行されるのではなく「最短で指定した秒数後に実行される」というだけである。

### Render Queue
* (?) 「Render Queue」という呼称・3階層モデルはHTML仕様上の正式な用語ではなく、レンダリングとイベントループの関係を理解しやすくするための簡略化されたモデルとして捉えた方がよい。仕様上は、イベントループの1回のイテレーションの中で条件を満たした場合に「レンダリングを更新する」ステップが実行される、という形で定義されている。
* コールスタックに処理がある間、ブラウザはレンダリングができない。つまり、コールスタックに処理がある間はRender Queueは待機している。
* ただし、Render Queueの処理はコールスタックには渡されない（この点がタスクキューと異なる）。
* 優先順位は Call Stack > Render Queue > Task Queue となる。
    * Call StackとRender Queueが空であれば、Task Queueの先頭から処理が1つだけ取り出され、Call Stackへpushされる。

### マイクロタスク
* マイクロタスクはマイクロタスクキューで管理される。
* マイクロタスクキューは、タスクキューよりも優先してイベントループによって確認される。
* Promiseの仕様、およびその基盤となる抽象的な「Job Queue」の概念自体はECMAScriptで定義されているが、それをいつ実行するかという具体的な「マイクロタスクキュー」としての処理手順（レンダリングとの兼ね合いを含む）はHTMLスペック側で定義されている。
    * (?) このため、一時期ブラウザごとにPromiseまわりの挙動が異なっていたことがある。

#### Promise
* Promiseはマイクロタスクを介して処理される。
* `Promise.resolve`はfulfilledなPromiseを返す。
* Promiseのcatch
    * `try 〜 catch`だとChromeのコンソールに`uncaught (in promise): 〜`が出てしまう場合がある。以下のようにすることでエラーが出なくなる。
    ```js
    var promise = Promise.reject('exception!!!').catch(function(e) { console.log(e) });
    ```
    * (参考)
        * https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/await
        * https://qiita.com/krmbn0576/items/e9bc5384b790df9e39ab
        * https://qiita.com/akameco/items/cc73afcdb5ac5d0774bc

#### async（async function）
* `async`をつけた関数はPromiseを返すようになる。
* 呼び出し後、（`await`に到達するまでは）即時同期的に実行される。
* 返り値
    * 関数内の返り値を`resolve`でラップして返す（`Promise.resolve(関数内の返り値)`を返す）。
    * 例外を投げている場合は`Promise.reject`で返す。
* `await`を呼んだ場合、呼び出し側の関数は実行が続く（このとき呼び出し側は、async関数から暗黙のPromiseを受け取っている）。

#### await（await演算子）
* Promiseが解決され、解決された値が返る（resolveされる）のを待つ。
    * つまり、Promiseが呼んだマイクロタスクがスタックにpushされて完了するまで待つ。
* `await`に続く値がPromiseでなかった場合は`Promise.resolve(値)`に変換される。
* 返り値は解決された値。
* `await`は基本的に`async`をつけた関数内でしか使えない。
    * ただし、ES2022からはモジュールのトップレベルで`async`をつけずに`await`を使う「トップレベルawait」がサポートされている。
* TIP: Promiseを一旦別の変数に入れることで、複数の非同期処理を並行して開始しつつ、後でまとめて待つことができる。
    ```js
    const axiosA = axios.get("./a"); // 1. マイクロタスクが投げられる
    const axiosB = axios.get("./b"); // 2. マイクロタスクが投げられる
    const axiosC = axios.get("./c"); // 3. マイクロタスクが投げられる

    const a = await axiosA; // 4. Promiseが解決される（マイクロタスクが完了する）のを待つ
    const b = await axiosB;
    const c = await axiosC;
    ```
    * (参考) https://makky12.hatenablog.com/entry/2020/01/11/175931

#### then（Promise.prototype.then()）
* (参考) https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Promise/then
* 最大2つの引数を取り、Promiseが解決した場合と拒否した場合それぞれのコールバック関数を渡せる。
* Promiseが解決/拒否されると、それぞれのコールバック関数が「非同期」に呼ばれる（マイクロタスクとして登録される）。
    * 例: fulfilledなPromiseに対して`then`を呼ぶと、引数のコールバック関数がマイクロタスクとして登録される（`Promise.resolve().then(コールバック関数)`）。
* 返り値はPromiseを返す。そのため、さらに`then`などでチェーンできる。
    * 値を返した場合、`then`が返すPromiseはその返り値でresolveする（何も返さなかった場合はundefinedとしてresolve）。
    * エラーを投げた場合、`then`が返すPromiseはその値でreject（拒絶）される。

#### さらに学習する場合の参考
* https://qiita.com/toshihirock/items/e49b66f8685a8510bd76
* https://www.tohoho-web.com/ex/promise.html
* https://azu.github.io/promises-book/#introduction
* awaitやasyncについての実験記事: https://zenn.dev/uhyo/articles/return-await-promise


## JavaScriptでマルチスレッドを実現する方法
* Web Worker（ブラウザ）やNode.jsの`worker_threads`モジュールを使う。
    * (?) Node.jsには`cluster`モジュールもあるが、これは複数のNode.jsプロセスをforkして分散させるマルチプロセスの仕組みであり、スレッドではない点に注意。Web Workerに対応する「実際にスレッドを使う」仕組みはNode.jsでは`worker_threads`である。
* (IMO) これらは厳密にはJavaScript自体の機能というより、JavaScriptの外部の仕組みを使ってマルチスレッドを実現しているものと捉えられる。


## window.locationの非同期性
* `window.location.replace`や`assign`
    * (参考) https://kokudori.hatenablog.com/entry/20120919/1348047725
    * `window.location.replace`は非同期に行われるため、その間に次の処理が進んでしまい、画面遷移前に意図しない描画（例: `this.create = true`によるDOM更新）が発生することがある。
    * `await`もできないため、リダイレクトする分岐の場合はそもそも描画がされないよう、後続処理を`else`句にまとめる、といった対応で回避した。
