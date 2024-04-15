- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)



# ドキュメント
* https://docs.flutter.dev/reference/flutter-cli
* https://dart.dev/tools/pub/cmd
* Flutterのコマンドは内部的に Dartのコマンドを利用している。

# グローバルオプション
* --help(-h)
* -v, --verbose
* -d, --device-id 
    * 実行環境を指定
* --version

# ヘルプ
* `flutter --help`

# プロジェクト作成
* `flutter create`
    * `flutter create --org com.example your_project_name`
    * サンプルプロジェクトから作成
        * `flutter create --sample=widgets.PageStorage.1 mysample`

# クリーン
* `flutter clean`
* build/と.dart_tool/ ディレクトリを削除

# pub系コマンド
* `flutter pub get`
    * pubspec.ymlに基づいてパッケージをダウンロード・更新
    * pub getコマンドはアプリが依存するパッケージを判断し、それをキャッシュフォルダへ配置する。
    * SDKの場合はこのキャッシュフォルダはインストール先のフォルダ内（~/flutter 等）、外部パッケージの場合は~/.pub-cacheに入っている
    * 参考
        * https://www.cresc.co.jp/tech/java/Google_Dart2/development/pub/pub.html
* `flutter pub add`
    * このコマンドは`dart pub add`と同じ処理をしている。
    * なお、`dart pub add 〜`　は  pubspec.yamlにパッケージを追加して　`dart pub get`を実行することと等価である。
* `flutter pub remove`
    * `flutter pub remove パッケージ名`
* `flutter pub run`
    * 各パッケージで実装されたコマンドを実行する。
    * 例
        * `flutter pub run flutter_native_splash:create`

# デバイス確認
* `flutter devices --machine`
    * 現在動作できるシミュレーターや端末を確認

# 実行
* `flutter run -d デバイス`
* pub get、ビルド、インストール、実行まで全てを行う。ホットリロードなどのショートカットも表示される
* モードの指定
    * --debug, --release, --profile
    * デフォルトでは--debugとなっている。
* --[no-]build
    * ビルドを行うか否かを指定。
    * デフォルトは--build
* --[no-]hot
    * ホットリロード機能をONで利用するか否かを指定。
    * デフォルトは--hot
* --target
    * エントリーポイント
    * デフォルトではlib/main.dart
* --dart-define-from-file
    * dart-defineのファイルを指定できる。
* 表示されるURLをブラウザから開くとインスペクタにて確認が可能。
    * (参考)(IME) このURLは、例えば「 http://127.0.0.1:9102?uri=http://127.0.0.1:65045/AH4QroZkvO8=/」のようになっているが、
        * 「http://127.0.0.1:9102」は、flutter runが「dart devtools」によって起動したdevtool
        * 「http://127.0.0.1:65045/AH4QroZkvO8=/」は、起動したDart VMが提供するURL
        * 参考
            * https://zenn.dev/tatsuyasusukida/scraps/a740047660e72e


# ビルド、インストール
* `flutter build`
* `flutter install`


# デバイスのデタッチ・アタッチ
* https://docs.flutter.dev/add-to-app/debugging
* デタッチ
    * `flutter run`を実行した後に d を押下するとデタッチされる。
* `flutter attach --device-id (対象のデバイス)`
    * 再度デバイスをアタッチ
    * --device-id でデバイスを指定しない場合、先頭のデバイスが使われる？

# その他
* `flutter screenshot --out 出力先ファイル`  
    * 現在接続中のデバイスのスクリーンショットを撮る



