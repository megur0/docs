[TOP(About this memo))](../README.md) > [一覧(Flutter)](./README.md) > パッケージ開発



# TODO
* 詳細まで書く

# 公式
* https://docs.flutter.dev/packages-and-plugins/developing-packages
* https://dart.dev/guides/libraries/create-packages

# 基本
* パッケージ構成
    * pubspec.yaml
    * lib
* 種類
    * Dart package
        * Dartで書かれた一般的なパッケージ
        * Flutter frameworkに依存
    * Plugin package
        * Dartコードで記述されたAPIとプラットフォーム固有の実装を組み合わせたパッケージ
    * FFI plugin package
        * Dart コードで記述された API と、Dart FFI ( Android、iOS、macOS ) を使用するプラットフォーム固有の実装を組み合わせたパッケージ
* 作成
    * `flutter create --template=package hello`
* パッケージのpubspec.lockはignore対象となる
    * https://dart.dev/guides/libraries/private-files#pubspec-lock
    * アプリケーションコートど異なり、ライブラリの場合はpubspec.lockは管理対象外となる。
    * これはパッケージはlockされたバージョンではなく、pubspec.ymlで指定されたバージョンすべてで動作する必要があるため、と考えられる。
        * 仮にlockファイルによって依存先のパッケージのバージョンが固定されてしまうと容易に複数のパッケージで依存のconflictが発生してしまう。

## パッケージの公開
* パッケージの公開
    * https://docs.flutter.dev/packages-and-plugins/developing-packages#publish
    * https://dart.dev/tools/pub/publishing#publishing-is-forever
        > Remember: Publishing is forever
        * 一度公開したパッケージは削除できない点に注意
        * 廃止済みとしてマーク(パッケージがアクティブなメンテナンスを受けていないことを開発者に通知)することは可能
    * パッケージの Web ページのコンテンツ(pub.dev/packages/パッケージ名)に影響を与えるファイル
        * LICENSE
        * README.md
        * pubspec.yaml
    * Google アカウントが必要
    * 検証済みの発行者になる (推奨)
        * https://dart.dev/tools/pub/publishing#verified-publisher
    * テスト公開
        * `dart pub publish --dry-run`
        * パッケージがpubspec 形式とパッケージ レイアウト規則に従っていることを確認
        * 公開する予定のすべてのファイルを表示
    * 公開
        * `dart pub publish`
        * 以下のファイル・ディレクトリを除いてすべてが公開される
            * ドット ( .)で始まる隠しファイルまたはディレクトリ。 
            * .pubignoreまたは.gitignoreファイルで無視されるようにリストされているファイルとディレクトリ
* プレリリース
    * セマンティックバージョニングにしたがって、suffixをバージョンへつける。
    * 例
        * 2.1.0 -> 2.1.0-dev.1
    * pubは安定版が利用可能な場合はそちらを優先するため、ユーザーがプレリリースでテストしたい場合はpubspec.ymlで^2.0.0や^2.1.0ではなく ^2.1.0-dev.1 を指定する。

# プラグインパッケージ
## フェデレーションプラグイン
* さまざまなプラットフォームのサポートを個別のパッケージに分割する方法
* プラグインパッケージを作成する場合は、基本的にフェデレーションプラグインによって作成する。
* フェデレーションプラグインには以下のパッケージが必要となる。
    * アプリ向けパッケージ
    * プラットフォーム パッケージ（ios, Android, windows 等）
    * プラットフォームインターフェースパッケージ

# pub.devにパブリッシュされていないパッケージを利用する
* https://docs.flutter.dev/packages-and-plugins/using-packages#dependencies-on-unpublished-packages
* pathでローカルのフォルダを指定できる。
* gitでgitリポジトリを指定できる。
    * urlでリポジトリを指定
    * pathでリポジトリ内のパスを指定できる。
    * refではコミットハッシュやブランチ、タグ等を指定できる。
        * 最新版を常に取得するにはref: HEADと指定する。（指定しない場合は初回にダウンロードしたコミットハッシュのバージョンを使用し続ける?）
    * https://stackoverflow.com/questions/54022704/how-to-add-a-package-from-github-in-flutter

# その他
## plugin_platform_interface
* https://pub.dev/packages/plugin_platform_interface
* implementsではなくextendsを強制させる仕組みを提供
    * 該当のクラスは、PlatformInterfaceというinterfaceをextendsすることでこの仕組みを利用できる。
* パッケージベンダーが新しいAPIを既存のクラスに導入した際、利用者がそのクラスをimplementsしているとコンパイルエラーとなってしまうため。
* それを回避するためにパッケージベンダーは該当のクラスにplugin_platform_interfaceを継承させることで、もし開発者がimplementを使用した場合に動的にエラーを出力することができる。
    * 仕組みとしてはtokenを使っている。
* ただし、Dartのbaseキーワードの導入されたため、このパッケージは今後非推奨となる可能性がある。
    * https://pub.dev/packages/plugin_platform_interface#a-note-about-base
* 例えば permission_handlerパッケージ　PermissionHandlerPlatformクラスが利用していた。
## (参考)Semantic Versioning 2.0.0-rc.1
* https://semver.org/spec/v2.0.0-rc.1.html
