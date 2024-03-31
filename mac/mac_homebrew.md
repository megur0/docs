- [TOP](./README.md)
- [このメモについて](../README.md)


# Homebrewの用語
* formula
    * パッケージ定義を指す
    * ソースのURL、依存パッケージ、インストールスクリプト、テストスクリプト
    * 実体はパッケージ定義が書いてあるrubyファイルとなる。
* bottle
    * ビルド済みのバイナリ
    * kegと同じファイル構成をtar.gzにしたもの
    * 環境に合うbottleがformulaに含まれていれば，ビルドせずにインストール可能
* keg
    * インストールされたformulaもしくはbottle
    * 実行可能ファイルやライブラリの実体がここに置かれる。
    * `brew --cellar`によって確認すると配置場所は　/usr/local/Cellar
* tap
    * formulaを提供するリポジトリへの接続口
    * 例えば，`brew tap https://github.com/user/homebrew-repo` のように実行することでbrew installが可能なリポジトリを追加できるといった利用方法?
    * Homebrew/core はデフォルトでtapされている。ここに含まれるパッケージはHomebrewをインストールした時点で、brew installによってインストールできる。
        * https://github.com/Homebrew/homebrew-core
    * Homebrew/coreへ自分のformulaを取り込んでほしい場合は，PRを送って取り込んでもらう必要がある。
* cask
    * バイナリのみ提供されるパッケージ
    * たとえば、macOSアプリケーション等。
    * caskはHomebrew（自家醸造）できないもので，Homebrewとしては推奨していない。
        * caskはHomebrew/coreにマージされない。
    * インストール先はcellarとは異なる場所（/usr/local/CaskRoom）
* 参考
    * https://blog.ottijp.com/2020/05/23/homebrew/


# (参考)インストール先
```
$cat ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)" # $PATHに opt/homebrew/bin を追加する処理もここに含まれている?

$echo $PATH | sed -e 's/:/\n/g' | grep homebrew
/opt/homebrew/bin
/opt/homebrew/sbin

$which tree
/opt/homebrew/bin/tree

$ls -l $(brew --prefix)/bin
lrwxr-xr-x  1 xxxx  admin    29 11 21 13:22 tree -> ../Cellar/tree/2.1.1/bin/tree
...
```

# インストール
* インストール
    * https://brew.sh/
    * Next steps〜で表示されるコマンドの実行を忘れないようにする。
* アンインストール
    * https://github.com/homebrew/install#uninstall-homebrew

# update
* brew update
    * Homebrew自体のアップデート
    * 最新のパッケージ情報がアップデートされる
        * パッケージ自体がアップデートされるわけではない。
* デフォルトでは，前回のupdateから300秒経過している場合はbrew updateが自動で行われる。
* 参考
    * https://blog.ottijp.com/2020/05/23/homebrew/

# upgrade
* パッケージをアップデートする。
* `brew upgrade`で全てのパッケージ、Homebrew本体をアップデートする。
* brew upgradeで upgradeさせたくないパッケージを設定することができる。
    ```
    $ brew pin <formula>
    $ brew unpin <formula>
    ```

# 各種コマンド
* `brew doctor`
    * Homebrewによってインストールしているパッケージ内で異常がないか診断
    * 〜ready to brewと出力されれば問題ない。
* `brew config`
* `brew list(ls)`
    * インストールしたパッケージ一覧
    * `brew list --versions`
* `brew install パッケージ名`
* `brew uninstall パッケージ名`
* `brew update`
    * homebrewのみアップデート
* `brew upgrade`
    * Homebrew、全パッケージアップデート
    * パッケージ名指定も可能
* `brew search パッケージ名`
* `brew cleanup`
    * インストールしている古いパッケージを消去
* `brew deps`
    * `brew deps --tree cocoapods`

# homebrew-cask  
* caskをインストールするためのコマンド。  
* インストール
    * 2.7.0以降は `brew install --cask パッケージ名`
    * 2.6.0より前は `brew cask install パッケージ名`



