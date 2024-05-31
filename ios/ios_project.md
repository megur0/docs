- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)



# Xcodeプロジェクトの構成
* プロジェクトは、Target, Project, Workspaceの３つによって構成される。
* プロジェクトを開く際は、Projectで管理している場合はProject、Workspaceで管理している場合はWorkspaceから開く。
    * `open hoge.xcodeproj`
    * `open hoge.xcworkspace`
    * xedコマンドで開く場合
        * xedコマンドは、hoge.xcworkspaceを優先的に開く
        * ワークスペースやプロジェクトフォルダがあるディレクトリ内で `xed .`
        * もしくは `xed (ワークスペースやプロジェクトフォルダがあるディレクトリ)`
* 注意点としてCocoaPodsはxcworkspaceを利用する必要がある。
* Target
    * Product/Binary（例えばライブラリやアプリケーション)がどのようにビルドされるかを記述するもの
    * ビルド・実行時には常に特定の１つのTargetを指定する。
    * プロジェクトを作成すると自動で1つTargetが生成される。
    * Targetごとにビルド設定が含まれる。
        * したがってプロジェクトで共通のビルド設定を持たせて、Targetごとにサブセットの設定で上書きしたりファイルを持たせるといった利用が可能。
* Project(hoge.xcodeproj)
    * プロジェクトには、ファイル (コード/リソース)、設定、それらのファイルと設定からProductを構築するTargetが含まれる。
    * XcodeはProjectに基づいてTargetをビルドする
        * Projectのbuild setting は Targetのベースとなる。
        * Targetのbuild settingは Projectのbuild settingを上書きする。
    * Projectの中にはproject.pbxprojファイルが含まれる。
        * このファイルにはビルドで利用する多くのデータが含まれる。
    * 多くの場合はProjectで十分であり、Workspaceは必要ない。
    * ソースなどの依存関係がある場合にサブプロジェクトを利用する事ができる。
        * サブプロジェクトは個別に開くことも、親プロジェクト内で開くこともできる。
        * Project内のTargetは サブプロジェクト内のTargetに依存できるが、その逆はできない。
    * ライブラリ (サブプロジェクト) が複数のプロジェクトから使用される場合、Workspaceを利用して同じレベルのProjectとして配置すると扱いやすい場合がある。
        * Workspace内のProjectはお互いに同レベルの階層となる。（したがって相互に依存が可能。）
    * Projectの中に入っているワークスペース(project.xcworkspace/)は何か?
        * https://stackoverflow.com/questions/10956312/is-the-project-xcworkspace-file-important
            > Xcode always needs a workspace, and that the subfolder is some sort of "hidden" workspace that Xcode creates and maintains behind the scenes for you as long as you only work with the .xcodeproj.
        * これはXcodeが内部的に必要なため生成する"隠しワークスペース"のようなもの?
* Workspace (hoge.xcworkspace)
    * 複数のProjectをまとめる
    * CocoaPodsはWorkspaceを利用するため、CocoaPodsを利用する場合は.xcworkspaceを利用する必要がある。
    * Workspaceで管理している場合、Projectごとに開くこともできるが、その場合は依存関係が解決できないためビルドができない可能性がある。
    * contents.xcworkspacedata
        * Projectのリスト
    * xcuserdata/
        * 各ユーザーの設定を含むディレクトリ
    * xcshareddata/
    *    プロジェクトを共有するユーザーによって共有されるデータ
* 参考
    * https://stackoverflow.com/questions/21631313/xcode-project-vs-xcode-workspace-differences
    
# project.pbxproj
* 各ファイルや設定項目には一意に識別するためのuuidが割り当てされている。
    * https://stackoverflow.com/questions/22648347/project-pbxproj-hashing-for-files-what-hash-is-used-and-how
* buildSettings の 情報が含まれる。
* PBXProject
    * プロジェクト
* PBXFileReference, PBXBuildFile
    * ファイルへの参照?
* PBXVariantGroup
    * ローカライゼーションされているファイル?
* PBXGroup
    * Xcodeで開いた際のディレクトリ構造?
* PBX***BuildPhase
    * ビルドフェーズ
* (参考)
    * 下記の記事でproject.pbxprojの流れが図解されている。
    * https://qiita.com/yokomotod/items/02e395e99bb891d27f67

# xcconfig
* Build Settingsの内容が含まれるファイル
* Xcode上のBuild Settingsで設定を直書きした場合は、xcconfigよりもそちらの設定が優先される。
* したがってxcconfigの設定を反映するためには、Xcode上のBuild Settingsは項目ごとに$(inherited)を指定する必要がある。
* cocoapodsを使用している場合
    * cocoapodsが生成したConfigファイルの設定を読み込む必要がある
    * 各Configurationごとのファイルで生成されたファイルを#includeする。


# Assets.xcassets
* プロジェクト内の画像フォルダ
* デフォルトでプロジェクト中に生成される。

# CocoaPods
* https://cocoapods.org/
* iOSアプリ開発で使用するライブラリの管理を簡単にするためのツール
* インストールにはruby環境が必要
    * CocoaPodsは、rubyで書かれていてrubyのgemパッケージとして利用する。
* インストールはrubyのgem installコマンドでインストールする。
    * インストールは/usr/local/bin 等にされる。
    * gemはバイナリではなくRubyコードのため実行環境においてもruby環境が必要となる。
* Podfile
    * https://guides.cocoapods.org/using/the-podfile.html
        > The Podfile is a specification that describes the dependencies of the targets of one or more Xcode projects. The file should simply be named Podfile. 
* Podfile.lock
    * https://guides.cocoapods.org/using/using-cocoapods.html#what-is-podfilelock
        > This file is generated after the first run of pod install, and tracks the version of each Pod that was installed. 
* .podspec
    * 各Podライブラリに含まれる、バージョン情報等を記載したもの
    * https://guides.cocoapods.org/syntax/podspec.html
        > A specification describes a version of Pod library. It includes details about where the source should be fetched from, what files to use, the build settings to apply, and other general metadata such as its name, version, and description.
* ffi
    * rubyから C言語などを呼び出すライブラリ
    * CocoaPodsが依存している。


# Xcodeproj 
* RubyからXcodeのプロジェクトを作成したり編集したりするためのツール
* gemで提供される。


# Bundle configuration
* https://developer.apple.com/documentation/bundleresources/information_property_list/bundle_configuration
* CFBundleIdentifier
    * バンドルの一意の識別子。
* CFBundleShortVersionString
    * バンドルのリリースまたはバージョン番号。
* CFBundleVersion
    * バンドルの反復を識別するビルドのバージョン
* CFBundleName
    * ユーザーに表示されるバンドルの短縮名
* MinimumOSVersion
    * iOS、iPadOS、tvOS、watchOS でアプリを実行するために必要なオペレーティング システムの最小バージョン。
* CFBundleDevelopmentRegion
    * バンドルのデフォルトの言語と地域 (言語 ID)
* CFBundlePackageType
    * バンドルの種類
    * APPL: アプリ
    * FMWK: フレームワーク
    * BNDL: バンドル