---
title: "認証・認可（OAuth・OpenID Connect） - セキュリティ"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(セキュリティ)](./README.md) > 認証・認可（OAuth・OpenID Connect）


# ベーシック認証
* Wiresharkなどで平文でキャプチャできてしまう。
* (参考) https://qiita.com/STomohiko/items/52c21904d1753e47315b


# 2段階認証・2要素認証
## 2要素認証
* 例えば、PC以外にスマホなどの別デバイスでも認証すること。

## 2段階認証
* ID/PWに加えて、OTPによる認証を行うこと。

## 両者の違い
* 「2要素認証」と「2段階認証」は別物であり、同時に成立することもある。
* 同じ種類の認証要素を2つ組み合わせた認証は、本来は二要素認証とは呼ばないが、最近では「指紋認証＋静脈認証」のように「生体認証＋生体認証」など同じ認証要素を2つ組み合わせた場合でも二要素認証と呼んでいるケースもある(?)。

## TOTP（Time-based One-time Password）
* 時間に基づいて生成されるワンタイムパスワードの認証技術。RFC 6238「TOTP: Time-Based One-Time Password Algorithm」で定義されている。
* 代表的な実装にGoogle Authenticator、1Password、Authyなどがある。


# JWT（JSON Web Token）
* (参考) https://www.engineer-memo.net/20180716-4614
* (参考) https://developer.mamezou-tech.com/blogs/2022/12/08/jwt-auth/

## JWTのメリット
* セッションID方式では、リクエストの都度セッションIDをDBから検索する必要がある。
* JWTでは、秘密鍵や公開鍵でデコードすることで、DBにアクセスせずに検証できる。
* デメリットとしては、即時ログアウトがしづらい点が挙げられる。
    * セッションID方式であれば、DBからセッション情報を削除するだけでよい。
    * JWTの場合はブラックリストなどの仕組みで弾く必要がある。

## JWTとは
* RFC 7519で定義されている。
    * https://datatracker.ietf.org/doc/rfc7519/
    * JWTは、JWSおよび/またはJWE構造でエンコードされたJSONオブジェクトとして、クレームのセットを表す。
* Claim: JWS / JWTの中で使われる変数のこと。

## JWS（JSON Web Signature）とは
* 「JOSEヘッダー」「JWSペイロード」「JWS署名」で構成される。
    ```
    BASE64URL(UTF8(JWS Protected Header)) || '.' ||
    BASE64URL(JWS Payload) || '.' ||
    BASE64URL(JWS Signature)
    ```
* JOSEヘッダーの例: 以下のようなJSONをBASE64URLでエンコードしたもの。
    ```json
    {"typ":"JWT",
    "alg":"HS256"}
    ```
    * 上記をエンコードした値: `eyJ0eXAiOiJKV1QiLA0KICJhbGciOiJIUzI1NiJ9`

## JWS（JWT）を使った認証
* ログイン後、IDプロバイダーからJWS（IDプロバイダーの署名がされたもの）を発行してもらう。
* アプリ側の認証の際は、クライアントはこのJWSを渡し、アプリ側はJWTを検証する（これをIDフェデレーションという）。
* このように使われるJWTは「IDトークン」とも呼ばれる。GoogleやFacebookなどでの認証（いわゆるソーシャルログイン）の背景にはこの仕組みがある。
* メリット
    * IDプロバイダー側でID・パスワードを管理すればよく、アプリ側で管理する必要がない。
    * 通常、ユーザーログインにはIDとパスワードを管理するDBへのアクセスが必要だが、JWT認証ではその必要がない。
    * 認証処理をプレゼンテーション層で完結でき、DBアクセスが不要になることで認証処理の性能を向上できる。
    * クライアントからのリクエストの都度、認証済みのJWTを送信してもらい検証することでユーザーを識別できるため、サーバー側でセッション情報を維持する必要がない。
* デメリット
    * 通常のトークン認証方式と違い、ログアウトなどのタイミングで即時無効化するのが難しい。IDプロバイダー側でDBに保存しておけばログアウト実装は可能だが、それだとJWTのメリットを十分に活かせない。
    * そのため、IDトークンの有効期限は短くし（即時無効化に近づける）、リフレッシュトークンと組み合わせて使う対策が望ましい。それでもIDトークンの有効期限内は認証が通ってしまうため、本当に即時停止したい場合はアプリ側でブラックリストに入れるなどの対応が必要になる。
    * トークン認証方式であれJWTであれ、攻撃者に有効なトークンを窃取された場合に通信が成立してしまう点は共通。アクセストークンとリフレッシュトークンを併用したり、有効期間を短くしたりしてリスクを下げるのも両者で変わらない。

## トークン認証との違い
* トークン認証は、トークンの正当性確認のためにIDプロバイダーへの問い合わせが必要。
* JWTは公開鍵によって検証できる。
* 発行はいずれもIDプロバイダーが行う。

## その他
* JWTの実際の値は以下のような形式になる。
    ```
    eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIn0.iIr5BW1YfvKF3hK9_1tyf-hGvDs7G7mz8j59pOvi2sp5aX6_Zl0upHXLajbLL574UeB6yQqOxDAh0-WUPnqTLJxbtfIDe3Ni1GWcg4pKf9G0QVOw2EK4_PiSJyf1FAIouXrCgDGJRwFXwIRxlPrTfboCCo68hgXFBAMKLcJW7Pc
    ```
* LaravelのPassportはJWTを使っている。
    * (IME) IDプロバイダーとアプリの両方を同じサーバーで運用していたケースでは、JWTである必然性は薄かった。


# Bearer認証
* RFC 6750で定義。
* トークンを利用した認証・認可に使用されることを想定しており、OAuth 2.0の仕様の一部として定義されているが、その仕様内でHTTP全般での使用も認められている。
* (参考) https://qiita.com/h_tyokinuhata/items/ab8e0337085997be04b1


# トークンのセキュリティ
* PHPの`uniqid`は使わない方がよい。実質タイムスタンプに過ぎず、予測されうるため。
* 暗号学的に攻撃者が予測できてしまう値はNG。`random_bytes`のような暗号学的に安全な乱数生成器を使う。
* (参考: 徳丸浩氏によるトークン生成に関する指摘)
    * https://teratail.com/questions/60622
    * https://twitter.com/ockeghem/status/1215069575210946561


# OAuth 2.0
* そもそもの目的は、第三者にID/PWを管理させる必要をなくすこと。
    * (参考) https://qiita.com/TakahikoKawasaki/items/f2a0d25a4f05790b3baa
    > OAuth 2.0 とは、サービスのユーザーが、サービス上にホストされている自分のデータへのアクセスを、自分のクレデンシャルズ (ID & パスワード) を渡すことなく、第三者のアプリケーションに許可するためのフレームワークである。
* RFC 6749で規定されている。
* OAuthは認可（authorization）の仕様であり、認証（authentication）の仕様ではない。
    * ただし認可処理にはその一部として認証処理（「誰か」を確認する部分）が含まれるため、両者は混同されやすい。
* 認証（Authentication）: 誰であるか。
* 認可（Authorization）: 誰が誰に何の権限を与えるか。
* 認可サーバー: 認証を経て、アクセストークンを発行するサーバー。
* リソースサーバー: アクセストークンでユーザーの権限を確認し、データを渡すサーバー。リソースサーバーと認可サーバーは同一サーバーであることも多い。

## (執筆時点の情報) OAuth 2.1について
* OAuth 2.0（RFC 6749等）にその後のベストプラクティスを統合し、仕様を整理し直したものとしてOAuth 2.1がIETFで策定中。
* PKCEの必須化や、セキュリティ上非推奨とされたImplicit Flow等の廃止など、既存のセキュリティ関連仕様への追補（RFC等）で言われてきた内容を単一の仕様にまとめ直す位置付け。
* 2026年7月時点ではまだInternet-Draft（`draft-ietf-oauth-v2-1`）の段階で、正式なRFCとしては発行されていない。
    * https://oauth.net/2.1/
    * https://datatracker.ietf.org/doc/draft-ietf-oauth-v2-1/


# OpenID Connect
* 仕様: https://openid.net/specs/openid-connect-core-1_0.html
* OAuth 2.0規格の拡張仕様であり、「IDトークン」を発行するための仕様。
* (参考: OpenID Connectは実装コストが高いかどうかについての読み物)
    * https://oauth.jp/blog/2016/02/24/is-openid-connect-far-from-oauth2/

## response_typeの拡張
* (参考) https://qiita.com/TakahikoKawasaki/items/4ee9b55db9f7ef352b47
> RFC 6749 は認可エンドポイントというWeb APIを定義しています。このAPIは必須のリクエストパラメーターとしてresponse_typeを要求します。OpenID Connectは、このresponse_typeの仕様を拡張することによりIDトークン発行手順を定義しました。
> RFC 6749の時点ではresponse_typeが取りうる値はcodeかtokenのどちらかでしたが、OpenID Connectでは、id_tokenという新しい値を追加した上で、code、token、id_tokenの任意の組み合わせをresponse_typeに指定してもよいことにしました。加えて、noneという特殊な値も定義しました。

## IDトークン
* JWSのCompact Serialization形式。
* (参考) https://qiita.com/TakahikoKawasaki/items/8f0e422c7edd2d220e06

## 不明点（要調査）
* 認証画面を認可サーバー側に持たせるか、リソースサーバー側に持たせるかは、仕様上は明言されていない(?)。
    * GoogleのようなIDプロバイダーは、認証側が認証画面自体も提供しているケースが多い。
    * あるいは、画面自体は提供しなくても認証インターフェースを含むのがOpenID Connectの考え方だと思われる。
* APIサーバーがアクセストークンを発行し、フロントエンドと分離している構成であっても、それがOAuthの仕組みに則っていると言えるのかどうかは要検討(?)。
