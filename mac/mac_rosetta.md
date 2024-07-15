[TOP(About this memo))](../README.md) > [一覧](./README.md) >


# Rosetta
* Rosetta
    * 特定のアーキテクチャのバイナリを別のアーキテクチャのバイナリに変換することで互換性を維持する Apple の技術
* 各アプリケーションの「情報」を確認すると、そのバイナリの種類が確認できる。
    * Intelとなっているものがx86_64 となる。
    * Universalは x86_64 バイナリと ARM64 バイナリの両方に対応している。
    * アクティビティモニタでは、現在実行しているアプリについて、「種類」でアーキテクチャがわかる。
* アプリを開く際に、OSは、自動的に以下のように処理を行う。（Rosetta2がインストールされている前提）
    * Intel -> Rosetta2によってARM64へ変換される。
    * Univesal -> ARM64の方のバイナリが使われる（はず）。
* 参考
    * https://zenn.dev/suzuki_hoge/books/2021-07-m1-mac-4ede8ceb81e13aef10cf/viewer/3-rosetta2

# Rosetta2のインストール
*  Intelのみに対応したアプリを開くと、ポップアップが表示されインストールされる。
    * https://support.apple.com/ja-jp/HT211861
* コマンドの場合
    * `sudo softwareupdate --install-rosetta --agree-to-license`

# arm64, arm64e のどちらか確認
* `uname -m`
* arm64eがApple Siliconに該当する。
    * `man arch`の説明に書いてある。
    ```
    i386     32-bit intel
    x86_64   64-bit intel
    x86_64h  64-bit intel (haswell)
    arm64    64-bit arm
    arm64e   64-bit arm (Apple Silicon)
    ```


# 明示的に アーキテクチャを指定する
* `arch -x86_64 〜` のように 明示的に指定することで OSによる選択ではなく明示的に指定する。
    * この例では、内部でRosetta2によってARM64へ変換していると思われる。
* アプリによっては「情報」を開くと「Rosettaで開く」のチェックボックスがある。

# (IMO)アーキテクチャをどのように使い分ければ良いか?
* 基本的にARM64を使う。
* 各アプリも「Rosettaで開く」は基本的に使わない。
* 例外として、例えばx86_64のみ対応しているバイナリ等を使う際に`arch -x86_64 〜` によってx86_64などをARM64環境下で実行する。
    * 例
        * 以前、iOSアプリ開発で使うCocoapods では 依存するライブラリのffiがx86_64のみ対応していたため、Cocoapods自体もx86_64のバイナリのみインストールしていた。
        * その際は`arch -x86_64 pod install`によって実行していた。

# Terminal 
* Terminalにも「Rosettaで開く」がある。
* (IME)デフォルトのARM64で開くで良い。
