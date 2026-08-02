---
title: "Xcode Command Line Tools (CLT) - Mac"
---

[TOP(About this memo))](../README.md) > [一覧(Mac)](./README.md) > Xcode Command Line Tools (CLT)


## Xcode と Command Line Tools (CLT) の違い
* Xcode
    * `/Applications/Xcode.app/Contents/Developer`
    * 含まれるもの
        * clang, git, swift, xcodebuild, iOS Simulator, その他Xcode関連ツール
* Command Line Tools (CLT)
    * `/Library/Developer/CommandLineTools`
    * 含まれるもの
        * clang, git, make, その他最低限の開発ツール
        * GUIやシミュレータは含まれない。
* Homebrewなど、CLI経由でビルドツールを使うコマンドラインツールはCLT（もしくはXcodeに含まれる同等のツール）に依存している。

## xcode-select の役割
* 現在利用中の開発ツール（Xcode由来かCLT由来か）を確認する。
    ```
    xcode-select -p
    ```
    * 例: `/Applications/Xcode.app/Contents/Developer`
    * または: `/Library/Developer/CommandLineTools`
* 利用する開発ツールを切り替える。
    ```
    sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
    ```
    * 上記の例ではXcodeに含まれる開発ツールを利用する設定に切り替えている。

## xcode-select はXcodeとCLTのどちらに向けておくべきか(IMO)
* Xcode本体をインストールしているのであれば、基本的にXcode側に向けておく方が良い。
    * XcodeはCLTの上位互換に近い。clang/git/makeなどCLTに含まれるツール一式に加えて、iOS Simulatorやフルセットの各種SDKも含まれるため、「CLTだと足りない」というケースを気にしなくて済む。
    * XcodeとCLTを両方インストールした状態でバージョンがズレると、Homebrewから古いと判定されるといった不整合が起きやすくなる（詳細は後述）。Xcode側に統一しておく方がバージョン管理の手間が減る。
* CLT単体の方が良いケース
    * GUIやSimulatorが不要で、gitやclang程度のコマンドラインツールしか使わない場合。Xcode本体（数GB〜十数GB）を入れずに済み、ディスク容量・インストール時間を節約できる。
    * CIサーバーなど、GUIアプリを持ち込みたくない環境。

## 現在の環境を確認するコマンド
* 現在利用中の開発ツールの確認
    ```
    xcode-select -p
    ```
* CLTのバージョン確認
    ```
    pkgutil --pkg-info=com.apple.pkg.CLTools_Executables
    ```
    * 出力例: `version: 15.3`
* clangのバージョン確認
    ```
    clang --version
    ```
    * 出力例: `Apple clang version 17.0.0`
* Xcodeのバージョン確認
    ```
    xcodebuild -version
    ```
    * 出力例
        ```
        Xcode 26.2
        Build version 17C52
        ```
* 利用可能なアップデートの確認
    ```
    softwareupdate --list
    ```
    * CLTの新しいバージョンが提供されていれば `Command Line Tools for Xcode-16.4` のように表示される。

## CLTが古いことによる不具合(IME)
* Homebrewでパッケージをインストールしようとした際、CLTのバージョンが古いことが原因で以下のようなエラーが発生したことがある。
    ```
    Error: Your Command Line Tools are too outdated.
    Update them from Software Update in System Settings.
    ```
* `brew config` で `CLT:` の項目を確認すると、Homebrewが認識しているCLTのバージョンが分かる。
    ```
    brew config
    ```
    * 確認すべき項目
        ```
        CLT:
        Xcode:
        macOS:
        ```
    * (?)Xcode自体は最新でも、CLTのバージョンが古いままだとHomebrewから古いと判定される場合がある。

### なぜxcode-selectがXcode側を指していてもCLTのバージョンが問題になるのか(IME)
* `xcode-select -p` の出力は、あくまで「どの開発ツールディレクトリを使うか」という設定であり、CLTパッケージ自体のインストール状態・バージョンとは別で管理されている。
* macOSは、Command Line Toolsパッケージ（`com.apple.pkg.CLTools_Executables` 等）がインストールされているかどうかを、`pkgutil`のレシート（インストール受信記録）としてxcode-selectの設定とは独立に保持している。
* Homebrewの`brew doctor`/`brew config`は、xcode-selectの向き先だけを見ているわけではなく、`pkgutil --pkg-info=com.apple.pkg.CLTools_Executables`などを直接呼び出してCLTパッケージのバージョンを取得し、Homebrewが要求する最低バージョンと比較している。
    * そのため、`xcode-select -p`がXcode.app側を指していても、システムにインストールされているCLTパッケージ自体のバージョンが古ければ、Homebrewからは「古い」と判定される。
    * 今回のケース（`xcode-select -p`が`/Applications/Xcode.app/Contents/Developer`を指していたにもかかわらず、CLTのバージョン15.3が原因でエラーになった）はこれに該当する。
* (参考)
    * https://github.com/orgs/Homebrew/discussions/799
    * https://github.com/orgs/Homebrew/discussions/2773

## CLTの更新
### 方法1: softwareupdateコマンドで更新（推奨）
* `softwareupdate --list` で表示されたバージョン名を指定してインストールする。
    ```
    sudo softwareupdate --install "Command Line Tools for Xcode-16.4"
    ```
* 更新後は `pkgutil --pkg-info=com.apple.pkg.CLTools_Executables` でバージョンが上がっていることを確認する。

### 方法2: 再インストール
* 方法1で解決しない場合に試す。
1. 既存のCLTを削除する。
    ```
    sudo rm -rf /Library/Developer/CommandLineTools
    ```
2. 再インストールする。
    ```
    xcode-select --install
    ```
    * GUIダイアログが表示されるので、そこからインストールする。

## まとめ(IME)
* Homebrewの「Command Line Tools are too outdated」エラーは、CLTのバージョンが古いことが原因だった。
* `softwareupdate --install "Command Line Tools for Xcode-x.x"` で更新することで解消した。
* `xcode-select --switch` は「どの開発ツール環境（XcodeかCLTか）を使うか」を切り替えるコマンドであり、CLTのバージョン更新そのものとは別物。
* 既に `/Applications/Xcode.app/Contents/Developer` を利用している状態であれば、CLT更新後に `--switch` を改めて実行する必要はない。
