- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)


# Authentication エミュレータ
* セキュリティ上の理由から、署名なしの ID トークンを発行
    * このトークンは他の Firebase エミュレータ、または構成されている場合は Firebase Admin SDK によってのみ受け入れられる。
    * これらのトークンは、本番環境の Firebase サービスまたは本番環境モードで実行されている Firebase Admin SDK によって拒否される。
* エミュレートされたメール、メールリンク、匿名認証
* エミュレートされた電話 / SMS 認証
    * エミュレータの場合はreCAPTCHA フローと APN フローは無効になる？
    * エミュレータの場合はターミナルにSMSコードを出力。
    * 固定のログインコードでテスト用電話番号を定義する機能（Firebase コンソールを使用すると可能な機能）は、エミュレータでサポートされていない。
* 非対話型テスト
    * REST API を呼び出して確認コードを取得することができ、メールや電話認証などはスクリプト化可能。
* 多要素 SMS
    * エミュレータの場合はターミナルにSMSコードを出力。
* サードパーティ ID プロバイダ（IDP）認証、カスタム トークン認証
    * Google や Apple などの OpenID Connect プロバイダから得られる実際の認証情報は、Authentication エミュレータで受け入れられるが、OpenID Connect 以外のプロバイダから得られる認証情報はサポートされない。
* IAM
    * エミュレータは指定された Firebase セキュリティ ルールに従うが、IAM が通常使用される状況は（たとえば、Cloud Functions を呼び出すサービス アカウントの設定、権限の設定）、エミュレータを構成できないため、開発マシンでグローバルに利用できるアカウントが使用される。
* FDLに依存するリンク
    * モバイルではなくWEBで開かれる?
* Authentication エミュレータは、本番環境のレート制限や不正防止対策の機能を複製しない。
* ブロッキング関数
    * 本番環境では、beforeCreate イベントと beforeSignIn イベントの両方がトリガーされた後に、ユーザーはストレージに 1 回書き込まれる。
    * ただし技術的な制限により、Authentication エミュレータは、ユーザーの作成後とログイン後に 1 回ずつ、合計 2 回ストアに書き込む。
    * したがって新規ユーザーの場合、Authentication エミュレータでは beforeSignIn から getAuth().getUser() を正常に呼び出すことができるが、本番環境ではエラーが発生する。
* 実際にログインできるユーザーデータを生成するためには、パスワードを発行する必要がある
    * メールリンク認証とかは、あくまでもフローを確認するためのものであり、パスワードを発行しないと消えてしまう。
    * さらにパスワード認証で作成されたユーザーと、このメールリンクを送るために作成したユーザーのuidは別のものが生成され、customClaimsも引き継がれない。
    * 参考
        *  https://zenn.dev/tkow/scraps/8160da23277fb8
* cloud taskのような他のGCPサービスやAlgoliaのような外部サービスはモックする必要がある
    * 参考
        * https://zenn.dev/tkow/scraps/8160da23277fb8

必要は
# Firestore エミュレータ
* Emulator Suite UIでは、Firebase セキュリティ ルールの評価トレースなどを行うことができる。
* トランザクション
    * ロックが解除されるまでに最長で 30 秒かかる。
* インデックス
    * 複合インデックスの有無はチェックされない。
    * クエリが有効なものならすべて実行される。
    * したがって複合インデックスのチェックはコンソールプロジェクトへ接続する必要がある。
* 上限
    * たとえば、本番環境サービスではサイズが大きいことを理由に拒否されるトランザクションでも、エミュレータでは許容される場合がある。
## オフラインキャッシュ と エミュレータのデータ
* https://firebase.google.com/docs/emulator-suite/connect_and_prototype?hl=ja#connect_your_app_to_the_emulators
>注: Cloud Firestore エミュレータは、シャットダウン時にデータベースのコンテンツを消去します。Firestore SDK のオフライン キャッシュは自動的に消去されないため、エミュレートされたデータベースとローカル キャッシュの不一致を回避するよう、エミュレータ構成でローカル永続性を無効にすることもできます。Web SDK では、永続性はデフォルトで無効になっています。
* IMO: 実際、どうすれば良いのか？
    * エミュレータ側で起動時に`--import` と `--export-on-exit`を利用する。
        * ただこの対応のみでは、エミュレータ側のデータをクリアしたい時にオフラインキャッシュ側のデータが残ってしまう問題がある。
    * エミュレータ利用時は、firestoreのオフライン永続性を無効にする。
        * 上記の「エミュレータ構成でローカル永続性を無効にすることもできます」の説明が指す手段。
        * https://firebase.google.com/docs/firestore/manage-data/enable-offline?hl=ja#configure_offline_persistence
    * (参考)エミュレータ利用時は、オフラインキャッシュを都度クリアできないか？
        * clearPersistenceを使って毎回オフラインデータをクリアする方法も試したが、permissionのエラーが出てきてしまった。
            * `FirebaseFirestore.instance.clearPersistence();`
        * 調べたが、`FirebaseFirestore.instance.terminate()`してから実行する方法しか見つからなかった。アプリのライフサイクルにうまく組み込む方法が見つからなかったのでこの方法は断念。
            * https://firebase.google.com/docs/reference/android/com/google/firebase/firestore/FirebaseFirestore#clearPersistence()
            * https://stackoverflow.com/questions/63930954/how-to-properly-call-firebasefirestore-instance-clearpersistence-in-flutter


# Cloud Storage エミュレータ
* https://firebase.google.com/docs/emulator-suite/connect_storage?hl=ja

# Cloud Functions エミュレータ
* https://firebase.google.com/docs/emulator-suite/connect_functions?hl=ja
* エミュレータで可能な事
    * コーラブルな関数のエミュレート
    * バックグラウンドでトリガーされる関数のエミュレート
* Android
    * SDKから関数を呼び出すためにはnetwork_security_config.xml ファイルを設定する必要がある。
* Extensions から出力されるカスタム イベントのハンドラをテスト
    * この機能はあまり理解できていない。
    * Cloud Functions v2 で Firebase Extensions カスタム イベントを処理するために実装された関数の場合、Cloud Functions エミュレータは Eventarc エミュレータとペアになることで Eventarc トリガーをサポート
    * Firebase Admin SDK はイベントを Eventarc エミュレータに自動送信 -> Eventarc エミュレータは Cloud Functions エミュレータにコールバックし、登録済みのハンドラをトリガー
* シークレット値
    * デフォルトではエミュレータはアプリケーションのデフォルト認証情報を使用して、本番環境のシークレットにアクセスを試みる。
    * .secret.local ファイルを設定してシークレット値をオーバーライドすることができる。
* 関数に関するメモリやプロセッサの制限事項がエミュレータでは適用されない。
    * エミュレータを使用して関数を設計、テストした後、本番環境で制限付きのテストを実行して実行時間を確認すると良い。
* エミュレータは、エラー時の再試行をサポートしていない。
* typescriptの場合はコンパイル済みでない場合、エミュレータ立ち上げ時にエラーとなる。

# Extensions エミュレータ
* https://firebase.google.com/docs/emulator-suite/use_extensions?hl=ja
* 安全なローカル環境で拡張機能のインストールと管理を行い、請求額を最小限に抑えながらその機能を理解できる。
    * 拡張機能の中には、Google Cloud APIs を呼び出し、Local Emulator Suite 内にエミュレータが存在しないサービスにアクセスするものがあり、費用の請求が発生する可能性がある。
* 拡張機能のソースコードは、~/.cache/firebase/extensions にダウンロードされる。

