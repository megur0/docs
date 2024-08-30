[TOP(About this memo))](../README.md) > [一覧(Flutter)](./README.md) > アーキテクチャ



# ドキュメント
* overview
    * https://docs.flutter.dev/resources/architectural-overview

# Dartコード
* アプリ
    * Dartコード
    * 開発者が所有
* Framework
    * Dartコード
    * アプリを構築するための高レベルの API (ウィジェット、ヒット テスト、ジェスチャ検出、アクセシビリティ、テキスト入力など) を提供
    * アプリのウィジェット ツリーをシーンに合成

# Engine
> The engine’s C and C++ code are compiled with LLVM. 
* C/C++で書かれている。    
* フレームのペイント
    * 新しいフレームをペイントする必要があるたびに、合成されたシーンをラスタライズする役割
* Flutter のコアAPI を提供
    * グラフィックス、テキスト レイアウト、Dart ランタイムなどの低レベル実装を提供
* Dart VMを含む
    * iOS や Android などのプラットフォームデバイスの中で Engine をホストできる Dart VM（デバイスで動作するための最小部分）もここに含まれる。
* dart:ui として利用
    * このエンジンは dart:ui としてFlutter Frameworkから利用される。
* Engine -> 各プラットフォームのAPI の呼び出し
    * Embedder APIを使用して特定のプラットフォームと統合

# Embedder
* アプリをホストするOS と Flutter の間の接着剤
    * レンダリング サーフェス
    * アクセシビリティ
    * 入力ジェスチャ (マウス、キーボード、タッチなど)
    * ウィンドウ サイズ変更
    * スレッド管理
    * プラットフォーム固有の APIを公開
    * イベントループを管理
* Flutter アプリを開始すると、エンベッダーは以下の処理を行う
    * エントリポイントを提供
    * Flutter エンジンを初期化
    * Flutter が書き込みできるテクスチャを作成
* エンベッダーはプラットフォームに適した言語で書かれている
    * Android の場合は Java と C++
    * iOS と macOS の場合は Objective-C/Objective-C++
    * Windows と Linux の場合は C++ 
* 各プラットフォームのフォルダー（ios/, android/など）に入っている。


# Runner
* Embedder のプラットフォーム固有の API によって公開される部分を、ターゲット プラットフォームで実行可能なアプリ パッケージに構成
* flutter createでテンプレートが作成される。
* 各プラットフォームのフォルダー（ios/, android/など）に入っている。


# (IME)ビルド・実行の流れ
* 参考
    * https://qiita.com/kurun_pan/items/520d91b4f5da6f14345b
* 以下はflutter buildやflutter runコマンド実行時の内部処理のおおまかな流れ
* ※ やや推測も含まれるため、細かい箇所で誤りがあるかもしれない。
## Dartコード（アプリ・Framework）
* 参考
    * https://zenn.dev/tsuruo/articles/48909d22d49ffe
* 開発者が作成したアプリ自体の Dart ソースコードと、Flutter Framework (${install先}/packages/flutter) で提供されるライブラリのソースコードがビルドされる。
* Debug/Profile モード
    * DartVM 向けの中間言語のバイトコードにビルドされる。
    * 開発端末側でflutter_tools（${install先}/packages/flutter_toolsに入っているSDKのコマンドを実行するスクリプト）によって、ホットリロードのリクエストを受け取る度に中間言語へコンパイルする。このとき、前回のコンパイル結果を再利用するため、コンパイルされるものは差分のみとなる。
    * 生成された中間言語は iOS/Android上のDart VM へ行き渡ることで、変更結果が即時に反映される開発体験を実現している
        * VMに渡されるものは差分のみ？全量?
* Release モード
    * マシン語のネイティブコードにビルドされる。
    * 各Flutter アプリのプロジェクトディレクトリ直下にコンパイルされたコード（機械語）が置かれる。
## 実行バイナリ
* アプリを動かしたいターゲットの各OS向けの実行バイナリが作成される。
* ターゲットOS毎のToolchain (ビルド環境)にてビルドされる。
    * AndroidStudio や XCode, Clang 
    * 必要なツールがインストールされていない場合はエラーになる。
* 実行バイナリは下記で構成される
    * Engine（.so ファイル）
        * ダウンロードされた実行バイナリ（.so ファイル）
        * Dart VMも含まれる
            * ホスト上でエンジン（およびDartコードがコンパイルされて生成されたバイナリ?）を動作させる最小限のVM
            * この最小限のVMの機能に加えて、Debug/Profile モードの場合は中間言語を機械語に変換して実行する機能も含まれる。
        * 中核機能からプラットフォーム固有の処理の呼び出しは、Embeder APIを利用することで抽象化されている。
    * Embedder/Runner
        * Engineをリンクして実行するための各OSのプラットフォーム毎の処理
        * Embber APIによってプラットフォーム固有のAPIをラップしてエンジンへ提供
        * プラットフォーム固有の入力イベントなどをエンジンへ渡す役割もある?
        * プロジェクトディレクトリ直下の android/, ios/, macos/, linux/ に ソースコードがある。
## (参考)（例）iOSのシミュレーターや実機でFlutterコードが実行される流れ
* シミュレーターや実機での実行時には、iosフォルダに入っているソースコードがXcodeによってビルドされ、その実行バイナリがシミュレータ（実機）にインストールされる（という筆者の理解）
* このときDartコード（SDKのコード、アプリケーションコード）もコンパイルされる。
    * デバッグモードの際は中間言語
    * リリースモードの際は機械語（AOTコンパイル）
* iosフォルダに入っているものがEmbedder/Runnerであり、これをエントリポイントしてEngine（.soという実行バイナリ。Dart VMも含まれる）が呼び出しされる。
* Engine内のDart VMからDartコード（中間言語 or 機械語）が実行される。
* ホットリロードの際は、前回コンパイル結果を再利用して差分のみコンパイルして、シミュレータ（実機）上のVMへ送られ、それが即時に反映される。
* プラットフォーム固有の入力イベントなどEmbeder経由でで送られる？

