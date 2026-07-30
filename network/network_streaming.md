---
title: "ストリーム・リアルタイム通信 - ネットワーク"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(ネットワーク)](./README.md) > ストリーム・リアルタイム通信

## 所感（IMO）
* ラグが致命的ではないほとんどのWebサービスはポーリングで事足りそう。
* それに加えて、プッシュ通知はFCMなどの外部サービスを使うのが確実（あるいはロングポーリング）。
* ストリーミングはJavaScriptであれば、Streams APIが使えそう（フロントエンド、バックエンド(Node.js)）。
* WebSocketをWebシステムでわざわざ実装するユースケースはほとんど無さそう。実装コストもかなり高い。WebRTCはなおさら。
  * なお、Google MeetはWebSocketを一切使っていないらしい(?)。
  * TCPを利用した双方向通信の便利プロトコルとして、ブラウザ以外で使われていることの方が多い(?)。
  * Google、LINEのような大きな規模になってはじめて自前で実装するメリットがありそう。
* (参考) https://twitter.com/voluntas/status/1629665769675186179?cxt=HHwWhoDTgcOY3p0tAAAA

## 目的
* ストリーム：サーバーが非同期的にデータを流しつづけて、クライアントが順次処理を行うケース。
  * 具体例としては、Flutterの`Stream`クラスがわかりやすい。「非同期で流れてくるデータを受け取りながら処理するもの」である。UNIXの標準ストリームも同様。
  * 具体例：PUSH通知、動画ストリーミング。
* リアルタイムの相互通信：サーバーとクライアントがリアルタイムに相互に通信するケース。
  * 具体例：ライブチャット、オンラインゲーム。

## 知識
### 補足：ストリーミングとは
* 通信ネットワークを介して動画や音声などを受信して再生する際に、データを受信しながら同時に再生を行う方式。

### 標準ストリームとは
* UNIXおよびUnix系オペレーティングシステムや一部のプログラミング言語インタフェースにおいて、プログラムとその環境を実行前から接続している入出力チャネル。
* 標準入力(stdin)：デフォルトはキーボード。
* 標準出力(stdout)：デフォルトはディスプレイ。
* 標準エラー出力(stderr)：デフォルトはディスプレイ。
* リダイレクト（`>`や`>>`を使って書く）：標準出力の先を変更するもの。
* パイプ：標準出力を次のコマンドの標準入力に渡す。標準エラー出力は引き渡されない。

### 参考
* (参考) https://qiita.com/yuba/items/00fc1892b296fb7b8de9
* (参考) https://progzakki.sanachan.com/technology/server-push-technology/

## ストリーム、リアルタイムの相互通信に関する技術
### ポーリング
* クライアントからサーバに定期的に新着を問い合わせる。最も原始的かつ確実なやり方。

### ロングポーリング（“COMET”）
* ポーリングだが、問い合わせを受けたサーバは新着情報がなければレスポンスを返すのをしばらく保留する。
* そのあいだに新着情報が発生すれば即座にレスポンスを返し、一定時間経過したら何もなかったとレスポンスを返す。
* サーバ側でスレッド資源を浪費しないような工夫（非同期プログラミング）も必要。クライアント側はポーリングと変わらない。

### SSE(Server-Sent Events)
* ロングポーリングと同じく下り情報が何か発生するまでレスポンスを保留するところまでは同じ。
* 情報が発生するたびにレスポンスが完結するのではなく、長さ不定のレスポンスをだらだら返し続ける（情報が発生するたびにレスポンスの続きが来る）形。
* 下り情報が発生するたびにリクエスト・レスポンスが必要というロングポーリングのオーバーヘッド問題を解消する。
* クライアント側のプログラミングはポーリングと同じというわけにはいかない。

### WebSocket
* ロングポーリングにはパフォーマンス上の欠点があり、毎回の下り情報取得ごとにHTTPのヘッダが飛び交うという通信上のオーバーヘッドがある。高頻度・小サイズのデータがどんどん飛んでくるような状況だとオーバーヘッドが実データの数倍になる。
* オーバーヘッドなしに双方向通信できるようにしようという規格がWebSocket。
* Socketという名前にやや反し、バイト単位のストリームではなく電文単位のストリーム。
* HTTP（HTTPS）の体裁でリクエストするが、Upgradeというヘッダを付けることですぐにHTTPの枠外に飛び出し、一問一答というルールにとらわれずに小さな（最短で2バイトの）ヘッダだけ付けて電文をやりとりする。
* RFC6455で標準化。
  * プロトコルは`ws`または`wss`（SSL化したもの）と表記。
* 企業のHTTPプロキシが通してくれない場合がある。
  * (参考) https://qiita.com/yuba/items/00fc1892b296fb7b8de9
* 送達確認はWebSocketを使う側（アプリ層）で実装が必要。
  * (IMO) これがWebSocketのプログラミングを難しくしている感じがする。
  * (IMO) Goのライブラリも強いものが無く、C++やJavaもWebSocket関連のライブラリは欠点がありそうに見える。
  * (IMO) JavaScriptのSocket.ioはつながらない場合にポーリングで代替までやってくれるようなので、選択肢として良さそうに見える。
  * (IMO) 確実性を考えると「多少のラグがOKならポーリングの方がよいのでは」という結論になることも多そう。
  * (参考) https://qiita.com/yuba/items/00fc1892b296fb7b8de9
  * (参考) https://twitter.com/voluntas/status/1629755966530142208?cxt=HHwWgIDS-dWah54tAAAA
  * (参考) SSLアクセラレータを導入している環境ではWebSocketでpush通知を使う際の挙動が不安定になりやすい、という指摘もある： https://twitter.com/voluntas/status/1629777399142555648?cxt=HHwWgIDRrZv6kJ4tAAAA

### 補足：ソケット通信
* ソケット通信は、TCP/IPを利用する通信全般のことであるため、トランスポート層を指す。
* ソケットを利用してアプリケーション層のHTTP通信などが行われている。
* ソケットを使うとTCP/UDP上で動く（トランスポート層レベル）アプリケーションを開発することができるため、HTTPよりもコンパクトな通信でデータを送信することも可能。

### Connect（ConnectRPC）
* gRPCに代わる選択肢の一つ(IMO)。Buf社が開発しているRPCフレームワークで、Protocol Buffersのスキーマからコードを生成する点はgRPCと同様だが、ワイヤプロトコルとしてHTTP/1.1・HTTP/2の上でJSON/Protobufをやり取りでき、ブラウザからも追加のプロキシなしで直接呼び出せる点が特徴。
* connect-go: GoのRPCフレームワーク
  * (参考) https://future-architect.github.io/articles/20220623a/
* connect-es: TypeScript/JavaScript向け（旧connect-web）
  * (参考) https://future-architect.github.io/articles/20220819a/
* HTTP/2なしでもストリーミングができる。
* (?) 執筆時点では、AnthropicがSDKにConnectRPCを採用している、Buf社が社内でgrpc-goからconnect-goへ全面移行しているなど、当初の想定より広く採用が進んでいるようだ。以前ほど「敷居が高い」技術ではなくなってきていると考えられる(IMO)。
  * (参考) https://buf.build/blog/connect-a-better-grpc

### MQTT (Message Queuing Telemetry Transport)
* PubSubモデル（Publish/Subscribe）。
* これまでのHTTP1.xを使った仕組みは、クライアント・サーバーモデル。
  * クライアントとサーバーが直接コネクションを確立するため、莫大なクライアントとの通信に不向き。
  * スマートフォンなどは電波が安定していない場合も多いため、再送が発生するとさらにサーバー負荷が上がる。
* Brokerが間に入って、クライアントはBrokerとTCPまたはWebSocketで接続する（TCP/WebSocketとHTTPより負荷が低い）。
* LINEはこの技術を採用している。

### HTTP/2 Server Push
* HTTPの進化系として誕生したのがHTTP/2 Server Push。
* クライアントからのリクエストを待たずサーバーが送る仕組み。
* 開発者が期待していたような性能の改善が見込めず、HTTP/2で配信されているWebページのうちわずか1.25%しかこの機能を使っていなかったことなどを理由に、2022年にChrome（Chrome 106以降でデフォルト無効化）、2024年にFirefox（Firefox 132で完全削除）が対応を終了し、主要ブラウザでのサポートは完全に終了している。
  * (参考) https://asnokaze.hatenablog.com/entry/2020/11/13/001110
  * (参考) https://developer.chrome.com/blog/removing-push
* 後継としては、サーバーがレスポンス本文の生成前に先んじて`Link`ヘッダ等を返す「103 Early Hints」が挙げられる。

### WebPush
* HTTP/2の仕組みを利用してRFC8030で規格化。
* Push Serviceを中継し、クライアントとサーバーがPub-Subモデルで通信する。
  * クライアントとPush Serviceの間は、HTTP/2またはWebSocketが利用されることが多い。
* TwitterはWebPush+WebSocketに移行したという情報もある(?)。

### JavaScriptのStreams API
* (参考) https://developer.mozilla.org/ja/docs/Web/API/Streams_API
* (参考) https://zenn.dev/azukiazusa/articles/fetch-upload-streaming
* Node.jsでは、すでにStreams APIとほぼ同じ仕組みが昔から導入されており、細切れデータを扱うことができた。
* ブラウザの方には遅れてやってきた。

### 動画配信
* 動画配信だとHLS、WebRTC、RTMP、MPEG-DASHなどがある(?)。
  * RTMPは衰退していく傾向にあると思われる(IMO)。
  * WebRTC
    * TCPではなくUDPが採用されている。音声や映像はRTP。
    * プロトコルではなく技術。
    * 非常に低遅延だが、大規模配信は苦手。ブラウザからすぐに配信できる。
  * HLS(HTTP Live Streaming)
    * Appleが開発したStreaming Protocol。
    * iOSへ配信をする場合にはHLSでないとAppleの審査が通らない。
  * MPEG-DASH
    * HTTPを使った動画配信プロトコルはいくつもあり互換性がないため、HTTPプロトコルを使った動画配信プロトコルの国際標準規格として2012年4月にISO国際標準規格(ISO/IEC 23001-6)にて策定されたもの。
* YouTube Liveの例
  * カメラ配信者側はWebRTCを使うことでブラウザ単体で配信。
  * 視聴者側へはMPEG-DASHまたはHLS(HTTP Live Streaming)で配信。
* (参考) https://note.com/chitapapa/n/n49f5b7b635c4

## Firebaseとweb socket
* Firestoreは、クライアントとの同期に「WebChannel」という仕組みを使っている。通常はWebSocketで接続されるが、プロキシ等の環境でWebSocketが利用できない場合は、ロングポーリングにフォールバックする作りになっている。
* (参考) https://ably.com/topic/firebase-vs-websocket
* (参考) https://zenn.dev/cauchye/articles/20220127_kimurayu45z
* (参考) https://www.quora.com/Is-Firebase-an-MQTT
* FCMはHTTP v1 APIで実現している。
