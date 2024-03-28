- [TOP](./README.md)
- [このメモについて](../README.md)


# CocoaPods
* https://cocoapods.org/
* iOSアプリ開発で使用するライブラリの管理を簡単にするためのツール
* インストールにはruby環境が必要
    * CocoaPodsは、rubyで書かれていてrubyのgemパッケージとして利用する。
* インストールはrubyのgem installコマンドでインストールする。
    * インストールは/usr/local/bin 等にされる。
    * gemはバイナリではなくRubyコードのため実行環境においてもruby環境が必要となる。

# Xcodeプロジェクトの構成
* 参考
    * https://qiita.com/rockname/items/bbe73a001568f0f5e801
* TARGET
    * Targetとは、ビルドの結果得られる生成物のことで、iOS, watchOS, OSXなどがある。
    * Xcodeでプロジェクトを作成すると自動で1つ作成される。
    * TARGETを複数用意することで、テスト用、本番用などを切り替えてアプリをビルドすることができる。 
    * サーバー環境を複数用意する場合などは、TARGETも複数作成すると良い
    * どのTargetも build settings があり、ビルドするときは必ずいずれかのTargetを選択する必要がある。
* Project(.xcodeproj)
    * 複数Targetsをグループ化するものが Project 
    * Targetのベースとなる設定はProjectの build settings で行う。
    * Target固有の設定はそれぞれのTarget内の build settings で行う。
* Workspace (.xcworkspace)
    * 複数のProjectを同じレベルで束ねることができるのが Workspace
    * PackegeMangerにCocoapodsを使用している場合によく見られる
* Subproject
    * Projectの中に他のProjectを Subproject として組み込むこともできる。
    * 親ProjectをビルドすればSubprojectもビルドが走る(ビルドキャッシュ機構が動作するので、次に差分が生じるまでリビルドも生じない)。
    * 同じXcodeのウィンドウからどちらのProjectも編集でき、Targetも選択可能。
* WorkspaceとSubprojectの使い分け
    * 1つのメインプロジェクトがあり、そこからの参照しかない場合は Subproject として組み込む

# Xcodeproj 
* Rubyからxcodeのプロジェクトを作成したり編集したりするためのツール
* gemで提供される。

# xcconfig
* 参考
    * https://qiita.com/hirothings/items/7f6943db609ff88c10be
* Xcode の Build Settings をコードで設定するファイル。
* Xcode上のBuild Settingsで設定を直書きしている場合、そちらの設定が優先される。
* そのためxcconfigの設定を反映するためには、Xcode上のBuild Settingsは項目ごとに$(inherited)を指定する必要がある。
* cocoapodsを使っている場合、podsの設定の#includeが必要
    * cocoapodsが生成したConfigファイルの設定も読み込む必要があるため、podsの設定を各Configurationごとのファイルで#includeする必要がある。

