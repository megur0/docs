---
title: "Cookie・CORS・CSP・CSRF - セキュリティ"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(セキュリティ)](./README.md) > Cookie・CORS・CSP・CSRF


## クロスサイトスクリプティング（XSS）
* Webサイトの脆弱性をついて、悪意のあるスクリプトを埋め込んで実行させるもの。
* 「クロスサイト」と名前についているが、これは歴史的な経緯によるもので、現在はサイトを横断しない場合でもXSSと呼ばれる。
* https://jvndb.jvn.jp/ja/cwe/CWE-79.html
* https://ja.wikipedia.org/wiki/クロスサイトスクリプティング


## Cookie
* セッションはWebサーバー側にデータを保持する仕組み。
* 初回アクセス時にサーバーが新しいセッション領域を作りランダムなセッションIDを払い出し、ブラウザのCookieに保持させる。

### 原則: Cookieは発行されたドメイン配下のサイトからしか参照できない
* 例えばJavaScriptから参照できるのは、そのサイト（ドメイン）のCookieだけ（ただしサブドメイン間では共有できる）。
* もし他のサイトのCookieを参照できてしまうとしたら、それはブラウザ側の脆弱性であり大問題になる。

### サーバーとのやりとり
* サーバーは、HTTPレスポンスヘッダー「Set-Cookie」を送ることでブラウザにCookieを保存させることができる。
    * (参考) https://qiita.com/yassh/items/2088c6d026fb6b66806e
* 以降、ブラウザがHTTPリクエストを送る際、リクエスト先のドメインに紐づくCookieをCookieリクエストヘッダーで自動的に送信する。

### JavaScriptでの保存
* JavaScriptでは`document.cookie`にCookieをセットすることで、ブラウザにCookieを保存できる。

### 異なるドメインへのCookieの送信
* クロスオリジンのリクエストでもCookieを送りたい場合
    * `Access-Control-Allow-Credentials: true` を設定する。
    * `Access-Control-Allow-Origin` を明確に指定する（`Access-Control-Allow-Credentials: true` の場合、`Access-Control-Allow-Origin` にワイルドカードを指定するとブラウザがエラーを返す）。

### 異なるドメインへCookieが送信されてしまう脆弱性は発生しうるか
* (参考) https://at-virtual.net/securecoding/access-control-allow-origin%E3%81%AF%E3%81%A9%E3%82%8C%E3%81%BB%E3%81%A9%E8%84%86%E5%BC%B1%E3%81%AA%E3%81%AE%E3%81%8B%E8%84%86%E5%BC%B1%E3%81%AAcors%E8%A8%AD%E5%AE%9A%E3%81%97%E3%81%A6/
* おおよそ以下の条件が揃ったときにリスクとなる。新しいブラウザを使っている限りは影響を受けにくい。
    * 利用者が古いブラウザを使っている。
    * `Access-Control-Allow-Credentials: true` になっている。
    * `Access-Control-Allow-Origin: *` になっている（新しいブラウザであればここでエラーになるはず）。
    * Webサーバーが認証にCookieを使っている。
* 上記の条件が揃うと、攻撃者のサイトが利用者のアクセス時にリソースサーバーへ情報取得APIを投げると、利用者のCookieが自動的に付与されてAPIの認証も通ってしまい、利用者の情報が攻撃者のサイトに渡ってしまう可能性がある。

### Cookieとローカルストレージはどちらが良いか
* 一般的には、適切に設定されたCookie > ローカルストレージ。
* 設定が不十分なCookieにはCSRFのリスクがある。


## サードパーティCookie
* 広告などが利用している仕組み。
* Webブラウザで「www.example.com」を開いた際、「www.example.com」自身が保存するCookieがファーストパーティCookie、それ以外のドメインとして保存されるものがサードパーティCookie。
* サードパーティCookieはプライバシー侵害につながるとして問題視されてきた（あるサイトで見た内容が、別ドメインのサイトから追跡できてしまうため）。
* 2020年1月にGoogleが「サードパーティCookieを2年後に終了する」と発表したが、その後2021年6月24日には、Chromeでのサードパーティ Cookieサポートを2023年半ば〜後半の3カ月で段階的に廃止する見込みだと発表し直した（当初予定から1年以上延期）。
* その後さらに延期が続き、2024年にはGoogleが廃止計画自体を事実上撤回し、2025年4月には「Privacy Sandbox」経由でのユーザーへの選択プロンプト表示や、Chromeでのサードパーティ Cookie廃止自体を行わない方針を正式に発表した。
* (執筆時点の情報) 2026年現在
    * Chromeはサードパーティ Cookieをデフォルトでは廃止しておらず、既存のCookie設定機能を維持する方針。ユーザーがサイトごとに手動でブロック等を設定することは可能。
    * 一方、Safari（Apple）とFirefox（Mozilla）はすでにデフォルトでサードパーティ Cookieをブロックしている。
    * (参考) https://www.northbeam.io/blog/google-chrome-wont-be-deprecating-cookies----yet

### ITP（Intelligent Tracking Prevention）
* 2017年6月5日のWWDCでAppleが発表。
* Apple社のWebブラウザ「Safari」に搭載されたトラッキング防止機能で、機械学習を用いてCookieを判別し、ユーザーが望まないトラッキングを防止する。
* 具体的には、一定期間を過ぎたサードパーティCookieを削除することで、クロスサイトトラッキング（複数の異なるサイトを経由したユーザーの移動を追跡・分析すること）を抑制する。
* リターゲティング広告などが影響を受ける。

### サードパーティCookieは技術的にどう実現されているか
* 例えば広告配信などで、現在閲覧中のサイトとは異なるドメインのJSファイルを読み込ませ、それによってCookieへの書き込みを行う（あるいは、そのドメインからのレスポンスヘッダー「Set-Cookie」にセットすれば、ブラウザが自動的にCookieにセットする）。
    * このJSファイルを配信するドメイン側ではCORSを許可しておく必要がある。
* このJSファイルから、現在閲覧しているサイト自身のCookieを読み取ることはできない。
* しかし、このときに作成されたCookieは、発行元ドメインとの通信では（削除しない限り）常にヘッダーに載せて送信される。
* そのため、発行元ドメインにログインした時点で、このCookie情報と個人アカウントが紐づけられることになる。

### (参考) Googleのプライバシー対応の年表
サードパーティCookie以外も含めた、モバイル領域でのGoogleのプライバシー対応の経緯。
* 2021年: 「Google Play Services」のアップデートを発表し、「Android Advertising ID（AAID）」に対する広告主のアクセスを制限。
* 2023年2月14日: Android向けプライバシーサンドボックスのベータ版をリリース（発表から約1年後）。
* (参考)
    * https://www.exchangewire.jp/2023/04/03/what-the-pivot-to-privacy-has-meant-for-mobile/
    * https://www.nikkei.com/article/DGXZQOGN2305Y0T20C24A7000000/
    * https://news.yahoo.co.jp/articles/cf92d6636d142f37dc2ea342cc5f4cb6c2817fa9


## クロスドメイントラッキング
* (参考) https://www.mussyu1204.com/wordpress/it/?p=585

### セッションIDをパラメータに載せて引き継ぐ方法
* Cookieは同じドメイン間でしか共有できない（サブドメイン間では共有できる）。
* 例えばアクセス解析ツールなどは、タグを埋め込むと解析サービス側のサーバーがセッションIDを払い出し、それがCookieに保存される。ただし別ドメインに遷移するとセッションは途切れてしまう。
    * そのため、別ドメインへのリンクにセッションIDを埋め込むことでクロスドメイントラッキングを実現している(IMO)。

### サードパーティCookieを使う方法
* ただし、近年は禁止の流れに進んでいる。


## CORS（Cross-Origin Resource Sharing）
* TODO: CORSのリクエストの流れやヘッダーについてさらに学習する。
    * https://developer.mozilla.org/ja/docs/Web/HTTP/CORS
* (参考) https://qiita.com/att55/items/2154a8aad8bf1409db2b
* CORSとは
    * Web アプリケーションに対して、別オリジンのサーバーへのアクセスをオリジン間HTTPリクエストによって許可できる仕組み（JSからのAjax通信、XMLHttpRequestやFetch APIなどを想定）。
    * Same-Origin Policy（同一オリジンポリシー）によってデフォルトではブラウザ側で禁止されているため、通信を受けるサーバーサイドで許可する必要がある。
* オリジンとは
    * origin = protocol + domain + port number
    * ドメインとの違いは、プロトコルとポート番号も含む点。
* Same-Origin Policy（同一オリジンポリシー）
    * オリジン間のリソース共有に制限をかけるポリシーで、以下のような脆弱性を防ぐことを目的としている。
        * XSS（Cross Site Scripting）
        * CSRF（Cross-Site Request Forgeries）
        * ただし、XSS自体を防げるわけではない点には注意（リソース共有を許可しているサイト自体に脆弱性があれば、CORSの設定があっても意味がない可能性が高い）。
* 実装例
    * クライアントサイドでは特にやることはなく、異なるオリジンへのリクエストには自動的に`Origin`ヘッダーが付与される。
        * `Origin: https://trusted-one.co.jp`
        * XHRは特にやることはないが、Fetch APIでは`mode: cors`を設定する必要がある。
    * サーバーサイド
        ```yml
        Access-Control-Allow-Origin: https://trusted-one.co.jp  # 特定のサイトを許可する
        Access-Control-Allow-Origin: *                          # 全てのサイトを許可する
        Access-Control-Allow-Headers: "X-Requested-With, Origin, X-Csrftoken, Content-Type, Accept"  # 使うフレームワークにより異なるが、許可するヘッダーを定義する
        ```
    * プリフライトリクエストのレスポンスとして、許可するメソッドをレスポンスヘッダーに含める必要がある。
        * `Access-Control-Allow-Methods: PUT, DELETE, PATCH`

### CORSプリフライト
* https://developer.mozilla.org/ja/docs/Glossary/Preflight_request

### CORS設定だけでは不十分なケースがある
* プリフライトが発生しないリクエストの場合、リクエスト自体は届いてしまうことがある。
* つまり、プリフライトがない場合、許可されていないCORSに対するブラウザの制限は「レスポンス結果の読み取り」に対して働くため、ブラウザ側でレスポンスを解釈できないだけで、リクエスト自体はサーバーに到達してしまう(?)。
    * (参考) https://qiita.com/netebakari/items/41baa7e1d0b8d89f9d12


## CSP（Content Security Policy）
* (参考) CSPとCORSの違い: https://kotaroito.hatenablog.com/entry/2014/04/04/000533
* (参考) https://liginc.co.jp/blog/tech/639126
* インラインスクリプトも、nonceやハッシュ値によって制限できる。
    * https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Content-Security-Policy/script-src
* https://developer.mozilla.org/ja/docs/Web/HTTP/CSP


## CSRFとは
サイトを跨いだリクエストの偽造。正規のサイト以外から利用者を騙してリクエストを偽造して送らせるケースはCSRFと考えてよい。

Cookieと一緒にCSRFの話が出てくることが多いのは、Cookieの仕組み自体がCSRFの温床になりやすいため。Cookieはドメインに紐づいてリクエストへ自動的に付与されて送信される仕組み上、別のサイトからでもセッションを持ったPOSTやGETが実行できてしまう（もしこれができないと、別サイトへ遷移するたびに未ログイン状態になってしまい利便性が悪くなる）。

* (参考) https://blog.jxck.io/entries/2024-04-26/csrf.html
* ブラウザを経由せず直接URL（Web APIなど）を叩く攻撃は「クロスサイト」ではないため、CSRFには当たらない。

### CSRFトークン
昔からあるCSRF対策。サーバーがページを返す際に予測できないトークンをformに埋め込んでおき、受け取ったリクエストでそのトークンを検証する。これにより、外部からのリクエストはCSRFトークンを用意できないため防ぐことができる。

### 現在のCSRF対策
Originヘッダーのチェックと`SameSite`属性の設定で基本的にはカバーできる（CSRFトークンを別途使う必要は薄い）。
* (参考) https://speakerdeck.com/hiro_y/update-your-knowledge-of-csrf-protection?slide=21

### Originヘッダー
ブラウザはform送信の際にOriginヘッダーを付与している。これをチェックすることで、正規のサイトからのリクエストかどうかを判定できる。
* (参考) https://blog.jxck.io/entries/2024-04-26/csrf.html

### SameSite属性
Cookieを送信する範囲を指定できる属性。`Lax`（異なるサイトからの場合はGETのときだけ送信）に設定するのがちょうどよい落とし所になりやすい。ブラウザによってデフォルト値が異なるため、明示的に設定しておくことが望ましい。なお、GETに関してはSameSite=Laxでも送られてしまうため、GETリクエストでデータの変更などを発生させないよう注意する（通常はそのような実装はしないが）。
* (参考) https://speakerdeck.com/hiro_y/update-your-knowledge-of-csrf-protection?slide=21
* (参考) https://qiita.com/silane1001/items/2adba867f2c4e60ca8e5

### 別サイトへのGETやPOST
HTMLの`a`タグや`img`タグなどはGETできる。formはPOSTもできる。これらのレスポンス結果をJSから読むことはできない。JSでの`fetch`などは、CORSで許可されている場合のみ実行できる。ただし`document.forms[0].submit()`のようにformを自動送信したり、`window.location.href = "https://example.com"`としたりすることで、間接的にGETやPOSTを発生させることは可能。

### SPAではCSRFは可能か
認証が必要なURLの場合、ローカルストレージなどにアクセストークンを保存する構成であれば、その部分に関してCSRFは原理的に成立しない（ローカルストレージは別ドメインから読み取れないため）。一方、認証不要なURLに対しては、SPAかどうかに関係なくCSRFの可能性はある。
* (参考) https://qiita.com/lyd-ryotaro/items/d50a0fbab762af9a8714

### ローカルストレージへのアクセストークンの保存
基本的にはNGとされており、適切に設定されたCookie（`SameSite`、`HttpOnly`の設定に加え、Originヘッダーのチェックや適切なCORS設定も併用）の方が好ましいとされる。XSS攻撃が埋め込まれた場合、ローカルストレージへアクセスされて窃取されるリスクが上がるため。XSS対策を万全にしておくのは当然として、例えば外部ライブラリの脆弱性のように自分では制御しきれないリスクも考慮し、多層的に防御しておくのが基本になる。


## CSRFに関する考慮例
* あるシステムでCORSの`Access-Control-Allow-Origin`が`*`（任意のドメイン許可）になっているケースがあった。
* ただし、そのシステムではCookieによるセッション管理を行っておらず、認証はAuthorizationヘッダーのトークンで行っていたため、`Access-Control-Allow-Credentials`も無効であり、CSRFによる実害（Cookieの自動送信を悪用した攻撃）は発生しない構成だった。
* 一方、CORSの設定自体が緩い（`*`許可）ことは、それ自体が脆弱性となりうる点には注意が必要。
    * フィッシングサイトなどで利用者が認証情報やトークンを漏らしてしまえば、それを使って不正な操作をAPI経由でされてしまう。
    * 画面遷移を伴う一般的なサイトであれば、正規でないところからのリクエストであることをCSRFトークンによって検出できるが、API型のサービスの場合、フロントエンドからの正規リクエストと非正規のリクエストを完全に区別することは原理的に難しい。


## (実例) OAuth連携（Slack等）でのstateパラメータ設計
外部サービス（例: Slack）とのOAuth連携を自作サービスに組み込む際のCSRF対策の考え方の例。

### フロー
* 自作サービスの管理画面上（ユーザーがログインしている状態）で、ユーザーが「外部サービスのワークスペースへアプリを追加」のようなボタンを押す。
* 外部サービス側のOAuth認可画面が開く（この画面でユーザーが認可を行う）。
* 認可画面から自作サービスのサーバーへリダイレクトされ、その際にフィールドとして`code`を受け取る。
* 受け取った`code`を使って外部サービスのサーバーからアクセストークンを取得し、DBに保存する。
    * 以降はこのアクセストークンを使って外部サービスのAPIを利用できる。

### サーバー側で「どのユーザーか」を判定する必要がある
* 上記のフローでは、リダイレクト先で自作サービスのどのユーザーが認可を行ったのかを判定する必要がある。
* 外部サービスの認可画面へ遷移する際に付与できる「state」というフィールドを利用できる。

### stateに渡す値
* 単にユーザーIDを渡す方法
    * CSRFの脆弱性がある。
    * 他人にユーザーIDを窃取されると、そのユーザーIDを使って別のワークスペース上でなりすまし利用されてしまう恐れがある（攻撃者が無料でサービスを利用できてしまうなど）。
* ワンタイムトークンを発行してDBに保存し、照合する方法。

### その他考えられる攻撃
* ワンタイムトークンの窃取
    * 窃取されると、そのトークンを使って別のワークスペース上で利用されてしまう恐れがある。ただし有効期限を設けることでリスクはかなり抑えられる。
* `code`の窃取
    * `code`が窃取されても、クライアントシークレット（外部サービスのアプリに紐づくもの）がなければアクセストークンは取得できない。
* ワンタイムトークンのすり替え攻撃
    * 外部サービス→リダイレクトURLの間で、攻撃者が別アカウントで作成したワンタイムトークンにすり替えられると、攻撃者のアカウント側にアクセストークンなどの情報が保存されてしまう。
    * すぐに被害が出るものではないかもしれないが、サービスが提供する機能によっては攻撃の足がかりになりうる。対策としてはIPアドレスやUser-Agentの照合などが考えられる(?)。

### ワンタイムトークンの生成方法
* SHA-256でユーザーID＋ランダム値をハッシュ化する方法でよいと考えられる。
    * ユーザーIDは必須ではないが、予測されづらい値としてユーザーID＋ランダム値を使うのがわかりやすい。
* HMACやシークレットキーを使う必要は薄い(IMO)。途中で改ざんされた場合はDB上との照合で検知できるため、シークレットキーを使った改ざん検知までは不要なケースが多い。


## セッションハイジャックの例
* HTTP通信を盗聴してCookieからセッションIDを盗む。
* XSSでCookieからセッションIDを盗む。
* セッションフィクセーション
    * 攻撃者があらかじめ用意したセッションIDをURLパラメータに載せて利用者に踏ませる。
    * 攻撃者のセッションIDをBodyに載せてPOSTさせる。
* 対策
    * `HttpOnly`属性を設定する（JavaScriptからの取得を防ぐ）。
    * `Secure`属性を設定する（HTTPS通信のみで送信されるようにする）。
    * XSS対策を行う（入力値のエスケープ処理）。
    * セッションIDをURLパラメータに載せない。
    * `SameSite`属性を`Lax`にする（異なるドメインからのPOSTではCookieを送信させない）。
