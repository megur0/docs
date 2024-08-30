[TOP(About this memo))](../README.md) > [一覧(Mac)](./README.md) > コマンド・ショートカット


# (注)
* ここに記載のコマンドやショートカットは筆者がよく使うもの（or 今後、積極的に使っていけたらと考えているもの）となる。

# 画面・ウィンドウ操作
* アプリ選択
    * cmd + tab
* 作業スペース切り替え
    * ctrl + <--> 
* フルスクリーン、解除
    * cmd + ctl + F 
    * fn + f はChromeの場合は機能するが、Vscodeの場合は機能しない。
* 全てをしまう。
    * cmd + opt + H + M 
        * こちらは以下のキーの組み合わせ
* アクティブ以外をすべて閉じる
    * cmd + opt + H
* 最小化してDockにしまう
    * cmd + M 
* 隠す
    * cmd + H
* アプリケーションのウィンドウを全てDockにしまう。
    * cmd + opt + M

# Finder
* 隠しファイルを表示・非表示
    * cmd + shift + .

# Spotlight検索
* cmd + space
   
# ポップアップ
* キャンセル: esc
* 保存せずに終了（閉じるを実行時に出現する確認ウィンドウ）
    * cmd + del or cmd + c 
* https://github.com/microsoft/vscode/issues/80811#issuecomment-971023274

# ミラーリング
* cmd + fn + F1
* ディスプレイ接続時にメニューからではなくショートカットから実行できる。

# Chrome
* 日本語URLをURLエンコードせずにコピ-
    * （ショートカットではないが）先頭の「h」を除いてコピーすると早い
* サーチバーへ移動
    * cmd + l
    * URLをコピペしたいときや、再検索の際に便利?
* スクロール
    * space, shift + spece
* 翻訳
    * 翻訳のショートカットがあれば便利だが、デフォルト機能としては存在しない。

# 絵文字
* cmd + ctl + spece 
* 表示されないエディタもある

# terminal
* タブ移動
    * shift + cmd + <->
* 1行削除
    * cmd + l 
* ログインした日時を確認
    *  `last | grep console `

# ファイルのメタ情報の削除
* 確認
* `mdls ファイル名`。
    * ファイル名はファイルをterminalにドラッグすると早い。
* 削除
    * `xattr -c ファイル名`