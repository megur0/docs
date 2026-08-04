---
title: "キーボードショートカット - VSCode"
updated: 2026-08-02
---

[TOP(About this memo))](../README.md) > [一覧(VSCode)](./README.md) > キーボードショートカット


## トラックパッドとキーボードの使い分け
* トラックパッドにもキーボードにも、それぞれメリット・デメリットがある。
* 両方使えるようにしておき、状況に応じて直感的に使い分けることが効率化のために大事(IMO)。
* トラックパッドを実際には多用するものの、キーボード操作だけで完結できるスキルも持っておいたほうがよい(IMO)。

| | トラックパッド | キーボード |
|---|---|---|
| エディタ上で目的地までの移動 | 早い | 遅い |
| パネル間移動 | 遅い | 早い |
| 覚えること | 少ない | 多い |

## パネル表示・移動

### Zen mode
* `cmd + k` → `z`
* `esc`を2回押しても解除可能。

### ウィンドウの移動
* `ctrl + w`して選択。
* ただし、トラックパッドの方が早い場合も多い(IME)。表示しているものが多い場合に使う可能性がある程度。

### ウィンドウのリロード
* コマンドパレットを表示 → `Developer: Reload Window`
    * VSCodeで時々、すでに削除したファイルやプロジェクト外のファイルがProblemパネル上に表示され続ける事象が発生することがあり、その際にリロードすると復旧する(IME)。
    * ただし、実行中のデバッグやターミナル上のプロセスは終了してしまうため注意。

### エディタ ⇔ ターミナル
* ターミナルへ: `ctrl + ^`（たまに効かないことがある(IME)）
* エディタへ: `cmd + 1`（分割している場合は`cmd + x`）
* パネル表示・非表示: `cmd + J`

### エディタ ⇔ Problem
* `shift + cmd + M`

### エディタ ⇔ Debug Console
* `shift + cmd + Y`

### エディタ ⇔ ソースコントロール
* `ctrl + shift + G`

### サイドバー（エクスプローラ） ⇔ エディタ
* サイドバートグル: `cmd + b`
* サイドバーへ
    * サイドバーへ: `cmd + 0`
    * エクスプローラー: `shift + cmd + e`
    * 検索: `shift + cmd + f`
    * ソースコントロール: `shift + ctrl + g`
        * ソースコントロールは選択時にトラックパッドを使うことが多いため、ショートカットの出番はほぼない(IME)。
* ファイルを開いてエディタへ移動: `cmd + down`
    * `space`: 閲覧はできるがフォーカスが移動しないので、ほぼ使わない。
* ファイルを今のエディタの横に開く
    * `ctrl + enter`
    * `cmd`を押しながらダブルクリック。

### エディタ分割
* `cmd + 2`, `+ x`で分割。
* `cmd + 1`, `+ 2`, `+ x`で分割スペース間を移動（`opt + cmd + ←→`でも移動可能）。
* `cmd + b`でサイドバーを閉じる。
* `cmd + j`でターミナルを閉じる。
* `cmd + k`, `cmd + m`でエディタを拡げる・戻す。
* `ctrl + cmd + ←→`で現在のファイルを分割スペース間で移動できる。

## 検索

### 全体検索・置換
* `cmd + shift + F`

### コマンドパレット
* `cmd + shift + P`（Mac）
* ファイル検索の入力欄に`>`が入った状態（コマンド実行モード）になる。

## エディタ操作
* 公式: https://code.visualstudio.com/docs/editor/codebasics

### ファイル
* `cmd + p`でファイル名指定。
* ファイルを閉じる: `cmd + w`
* すべてのタブを閉じる: `cmd + k` → `cmd + w`
* 閉じたファイルを開き直す: `cmd + shift + t`

### 検索・置換
* `cmd + F`、`esc`
* `cmd + g` / `cmd + shift + g`: 前の検索結果 / 後の検索結果
* `cmd + opt + F`
    * 置換: `enter`
    * すべて置換: `cmd + enter`

### ジャンプ
* 参照: `F12`
* 参照元へ: `shift + F12`
* エラーへジャンプ: 次は`F12`、前は`shift + F12`
* 戻る: `ctrl + -`
* 進む: `ctrl + _`
    * Web検索では`ctrl + shift + -`と紹介されていることもあったが、自分の環境で確認した割り当てとは異なっていた(IME)。

### 情報表示
* 引数名を表示: `cmd + opt`
* 情報ポップアップを出す: `cmd + k` → `cmd + i`
* APIの説明: `cmd`を押しながら関数にフォーカスすると詳細が見られる。

### スクロール
* `pageUp` / `pageDown`（`fn + up/down`）

### タブ移動
* `option + cmd + ←→`

### マルチカーソル
* `cmd + opt + up/down`

### 移動
* 行の先頭・末尾: `cmd + ←→`
* 指定の行へ移動: `ctrl + g`
* ファイルの先頭・末尾: `cmd + up/down`
* 単語移動: `opt + ←→`

### 選択・コピー・貼り付け・挿入・削除・切り取り
* ブロック選択: `cmd + d`（連続で押すと他の箇所もマルチカーソルで選択できる）
* スマート選択: `ctrl + shift + ←→`を連続で押すと選択範囲が広がっていく。括弧などの範囲選択に便利。
    * https://stackoverflow.com/questions/37835012/select-everything-between-matching-brackets-in-vs-code
* ボックス選択: `shift + opt`を押しながらマウスでドラッグ。
* 行選択: `cmd + l`（連続で押すと複数行選択。複数行への操作に便利）
* 行コピー: `cmd + c`
* 行切り取り: `cmd + x`
* 行挿入: 上は`cmd + shift + enter`、下は`cmd + enter`
* 行削除: `cmd + shift + k`（`cmd + l` → `del`でもよい）

### リンク
* 対象の文字を選択した状態で、URL形式の文字列をペーストするとリンクフォーマットになる。
    * `http`のようなURL形式の場合は適用されるが、ディレクトリパス形式では適用されない(IME)。

### コメント行にする
* `cmd + /`
* `cmd + k` → `cmd + c`
* https://atmarkit.itmedia.co.jp/ait/articles/1806/22/news034.html

### オーガナイズ・フォーマット系
* 未importのものをimportする: `cmd + .`
    * 言語によっては、organizeImportsの際に自動importしてくれるものもある（Goはやってくれるが、Dartはやってくれない(IME)）。
* organizeImports: `opt + shift + o`
    * これを使うより、保存時に自動実行されるよう設定しておいたほうがよい(IMO)。

### コードリーディング
* 現在のカーソル位置を包含するクラスやメソッドは、VSCode上部のバーで確認できる。
* 折りたたみ
    * https://tetorachaos.com/vscode-folding
    * 現在のファイルを全部折りたたみ・全部展開（よく使う(IME)）
        * `cmd + k` → `cmd + 0`（折りたたみ）
        * `cmd + k` → `cmd + j`（展開）
        * 用途: ファイルで定義されているクラス一覧・その中のメソッドなどをトップダウンに見ていきたいとき。
    * 現在のカーソル箇所をネスト部分まで含めて再帰的に折りたたみ（よく使う(IME)）
        * `cmd + k` → `cmd + [`
        * 用途: 今見ているクラスのメソッドを全部折りたたんで一覧で見たいときなど。ファイル全体の折りたたみのほうがよく使うかもしれない(IME)。
    * 現在のカーソル箇所をネスト部分まで含めて再帰的に開く（よく使う(IME)）
        * `cmd + k` → `cmd + ]`
        * 用途: 全部閉じてしまったものについて、今の箇所だけまとめて開くために使う。
    * 現在のカーソル箇所を折りたたみ
        * `cmd + opt + [`
        * 連続で押すと外側もどんどん閉じる（ファイル全体の折りたたみまではいかない）。ネスト部分は閉じない。
        * 用途: 今いる箇所から移動せずに、上の階層へどんどん行きたいとき(IME)。
    * 現在のカーソル箇所を開く（たまに使う(IME)）
        * `cmd + opt + ]`
        * ネストしている部分までは開かない。連打しても、折りたたみ済みの箇所を戻していくわけではなく、現在箇所を開くのみ。
        * 開いてから戻す場合は`ctrl + -`を使うため、あまり出番はないかもしれない(IME)。
* 対のカッコへ移動(?)
    * Mac: `shift + cmd + \`
    * かっこだけでなく、内側で囲まれているブロックの位置にも移動してくれるようだ(IME)。

### サジェスト
* `cmd + i` または `ctrl + space`
    * サジェストの手動表示。
    * https://github.com/microsoft/vscode/issues/85643
    * Macの場合、`ctrl + space`は入力ソースの切り替えショートカットに割り当てられているため、システム設定の「キーボードショートカット > 入力ソース」からオフにするとよい。設定変更後もVSCodeでショートカットが反応しない場合は、VSCodeを再起動したほうがよいかもしれない(IME)。

## ターミナル・エクスプローラ操作

### ターミナル操作
* 前後のコマンドへ移動: `cmd + up/down`
* スクロール操作をキーボードとトラックパッドどちらで行うかは悩ましいが、今のところトラックパッドを使っている(IME)。
    * ターミナルの場合、カーソル移動のためにトラックパッドを使うことがないため、キーボード操作だけで完結できそうではある。`scrollPageDown/Up`を駆使する手もあるかもしれないが、MacBookのキーボードはショートカットのカスタマイズが必要なため、今のところやっていない。`pgUp`/`pgDown`キーが付いていれば使っていたと思う(IME)。

### エクスプローラ
* `space`で表示。
* `←`: ディレクトリを折りたたむ。
* `→`: ディレクトリをすべて開く。
* `cmd + ←`: ディレクトリをすべて折りたたむ。

## カスタマイズ

### キーボードで画面中央に寄せるには
* 拡張機能をインストールする方法もあるが、ビルトインコマンドをキーボードショートカットに割り当てるだけでも実現できた(IME)。
* https://stackoverflow.com/questions/49997089/vs-code-how-to-set-that-when-i-press-ctrl-up-or-down-arrow-the-cursor-move-w
* https://stackoverflow.com/questions/57436122/vs-code-keyboard-shortcut-to-move-cursor-to-the-center-of-current-screen-after
* https://stackoverflow.com/questions/41082702/how-to-center-the-editor-window-back-on-the-cursor-in-vscode

### （参考）Vim拡張について
* Vim拡張などを使う方法は、カスタマイズ性や専用環境への依存度が高くなりすぎるため、あまり好ましくないと考えている(IMO)。
* いくつかサイトを見てみたが、この結論は変わらなかった(IME)。
* (参考) https://dev.classmethod.jp/articles/vscode_file_operation_shortcut_setting/
* (参考) https://qiita.com/TomK/items/3b1f5be07d708d7bd6c5
* (参考) https://zenn.dev/nash/scraps/3a4b04690a4bbf
* (参考) https://qiita.com/taekari/items/2d7d6a009937fb927810

### ショートカット確認・設定
* JSONファイルを直接編集するのがおすすめ(IMO)。
    * コマンドパレットで`keyboard`と入力すると開ける（`keybindings.json`というファイル）。
    * もしくは、右上あたりのアイコンをクリックしても開ける。
    * (参考) https://qiita.com/keitean/items/04727aeb673d1822107e
    * `settings.json`と同様に、`keybindings.json`を`.vscode`の中に入れればワークスペースごとの設定ができる可能性がある(?)（未検証）。
* 既存のショートカットの検索はUIが便利。
    * `cmd + k` → `cmd + s`でUIから開く。
    * 例: `alt+up`、`collapse`、コマンドID`workbench.action.navigateBack`などで検索できる。
* 設定をリセット
    * `keybindings.json`内の配列を空の配列`[]`にする。
    * UIから行う場合: Sourceが「User」になっているものが自分で設定したショートカット。「User」で検索し、1つずつ右クリックして「Reset Keybindings」する。
    * (参考) https://qiita.com/UsagiLabo/questions/81a756197ad6c6ac32a8

## キーボードショートカットのトラブルシューティング
* コマンドパレットで「Developer: Toggle Keyboard Shortcuts Troubleshooting」を開くと、どのキーバインドが発火したかを確認できる。
* https://code.visualstudio.com/docs/getstarted/keybindings#_troubleshooting-keybindings
* https://stackoverflow.com/questions/46140269/how-to-debug-keybinds-in-visual-studio-code
* (参考) https://blog.solunita.net/posts/how-to-find-keybind-what-fired-in-vscode/

## その他
* ターミナル新規: `shift + ctrl + ^`
* mdファイルのプレビュー: `cmd + shift + v`
* コードの整形: `cmd + shift + P` → 「Format Code」で自動整形
* 設定を開く
    * 設定画面: `cmd + ,`
        * もしくはメニューから「Preferences」を開く。
    * 設定ファイル: コマンドパレットで`Preferences: Open Settings (JSON)`
        * プロジェクトごとの設定は`.vscode/settings.json`で設定できる。
