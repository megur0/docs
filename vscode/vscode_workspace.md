---
title: "ワークスペース・基本操作 - VSCode"
updated: 2026-08-02
---

[TOP(About this memo))](../README.md) > [一覧(VSCode)](./README.md) > ワークスペース・基本操作


## ワークスペース
* VSCodeは、現在開いているウィンドウ1つが1つの「ワークスペース」となる。
    * 何もしなくても、現在のウィンドウはワークスペースとなる。
* ワークスペースは保存できる。
    * File > Save Workspace As
    * 保存したワークスペースファイルから後で開き直せる。
* ワークスペースには任意の数のフォルダを追加できる。
    * File > Add Folder to Workspace
* メリット
    * 単体のフォルダの場合はそのフォルダを開けばよいのでそこまでのメリットはないが、複数フォルダを同じウィンドウで見たい場合に便利。
* デメリット
    * 各種パネルの表示がフォルダごとに分割されるため、1つのウィンドウの作業スペースが狭くなる。
    * `cmd + p` によるファイル検索が複数フォルダ分混ざる。

## ファイル作成
* ファイル名にディレクトリ名も含めてまとめて作成できる（例: `test/index.js`）。

## 検索対象からの除外ファイル
* VSCodeでは除外設定されたファイルは検索対象から外れるが、「files to exclude」のsettingアイコンをクリックすると含めることができる。
    * (参考) https://qiita.com/isoken26/items/7ee93352d40cec95e8d4
* `node_modules`や`vendor`等、`.gitignore`に書いてあるものは除外されている。
    * 検索に含めるには、検索欄右側の「...」から「files to include」に入れておく。
    * 設定自体を変更する方法もあるが、手間がかかるため割愛。

## 最近開いたファイルの履歴を削除
* コマンドパレット > `File: Clear Recently Opened`

## タスクの実行
* (参考) https://matsuo-san.hatenablog.com/entry/2020/04/02/183248
* tasks.jsonの作成
    * ツールバーの「ターミナル」→「ビルドタスクの実行」を押すと実行するビルドタスクがないと言われるので、そこをクリック→「テンプレートから`tasks.json`を作る」を選択し、「Others」あたりを選ぶ。この時、作業ディレクトリに`.vscode`フォルダが生成され、中に`tasks.json`が保存される。
* 実行
    * コマンドパレットで`Tasks: Run Task` > `tasks.json`で設定したコマンドを選ぶ。
* 定義済み変数
```
${workspaceFolder}          VS Codeで開かれたフォルダのパス
${workspaceFolderBasename}  VS Codeで開いたフォルダの名前（スラッシュ(/)なし）
${file}                      現在開かれているファイル
```
    * その他: (参考) https://qiita.com/ShortArrow/items/dc0c8cacd696154510f1
