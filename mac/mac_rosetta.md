- [TOP](./README.md)
- [このメモについて](../README.md)


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

# (IME)アーキテクチャをどのように使い分ければ良いのか
* IME
    * 基本的にARM64を使う。
    * 各アプリも「Rosettaで開く」は基本的に使わない。
    * 例外となるケースでは、明示的に`arch -x86_64 〜` で指定する。
        * 例
            * iOSアプリ開発で使うCocoapods では`pod install`の際、依存先モジュールがARM64に未対応のモジュールがある等でエラーが発生することがある。
            * その場合は、`arch -x86_64 pod install`することで、podの x86_64のバイナリの方を実行することで対応した。
## (IME)Terminal 
* Terminalにも「Rosettaで開く」がある。
* デフォルトのARM64で開くで良い。
## (IME)Homebrew
* Homebrewをインストールすると、使われているアーキテクチャごとにインストール先が変わる。
* ネットの記事をいくつか確認すると、Homebrewを２つのアーキテクチャ別々でインストールしておく記事が多い。（これらは古い内容？）
    * 参考
        * https://zenn.dev/suzuki_hoge/books/2021-07-m1-mac-4ede8ceb81e13aef10cf/viewer/6-shell-policy
        * https://blog.mksc.jp/contents/apple-silicon/
        * https://zenn.dev/junjunjunk/articles/4b230519d87de4
        * https://zenn.dev/ress/articles/069baf1c305523dfca3d
* 筆者としては、Homebrew自体は ARM64のみインストールで良いと思う。
    * 下記のdiscussionsとかみても、「HOMEBREW_PREFIX: /usr/local」ではなく「HOMEBREW_PREFIX: /opt/homebrew」が適切なように見える。
    * https://github.com/orgs/Homebrew/discussions/4397
    * https://github.com/orgs/Homebrew/discussions/417
* （参考）x86_64とarm64を併用してしまうと、例えば、下記のようにARM64環境下でインストールをして際に「依存するモジュールがarm64でコンパイルされていない」といったエラーが出た。
```
brew install cocoapods
...
Error: cocoapods dependencies not built for the arm64 CPU architecture:
  ca-certificates was built for x86_64
  mpdecimal was built for x86_64
```
* arm64のhomebrewにてインストールされる個々のパッケージについては、基本的に /opt/homebrew/bin 配下にインストールされる。
    * arm64に対応していないものは、/usr/local/binにインストールされる?
    * 両方にインストールされるモジュールもある？
