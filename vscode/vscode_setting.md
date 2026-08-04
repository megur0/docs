---
title: "設定 - VSCode"
updated: 2026-08-02
---

[TOP(About this memo))](../README.md) > [一覧(VSCode)](./README.md) > 設定


## 設定の三段階
* 設定は以下の三段階があり、上から優先される(はず)
    * workspace settings（JSON）
        * 現在開いている環境の設定
    * user settings（JSON）
        * ユーザーの設定
        * すべてのワークスペースに適用される
    * default settings
        * デフォルトの設定。編集不可
* コマンドパレットで `setting` と入力すると、上記のJSON、あるいはUIで開くことができる（もちろん直接ファイルを開いてもよい）。
    * UI側で更新すると、JSONの中身も更新される。
* 設定内容
    * 共通の設定は自分のユーザー設定を参照する。
    * 各ワークスペースや言語ごとの設定は、プロジェクト直下の `.vscode/settings.json` に入れている(IME)。
* キーボードショートカットの設定は[キーボードショートカット](./vscode_keybinding.md)を参照。

## settings.jsonの作成
必要に応じて以下の手順でワークスペース単位の設定ファイルを作成する。
```bash
mkdir .vscode && touch .vscode/settings.json
```
`.gitignore`に`.vscode`以下を追加しておく。

* 設定項目の参考
    * (参考) https://qiita.com/ayatokura/items/4301e0d1d8b339f722eb
    * (参考) http://soonav.com/?cat=4
* 設定例
```json
{
  "files.autoSave": "afterDelay", // 自動保存

  // 開いたときのウィンドウの初期サイズ
  "window.newWindowDimensions": "maximized",

  "editor.fontSize": 14, // デフォルトの文字サイズが大きすぎるため変更
  //"editor.cursorSurroundingLines":100,//カーソルが常に真ん中になる
  "editor.wordWrap": "on", // 長い行を折り返して表示する
  "editor.minimap.renderCharacters": false, // ミニマップに文字ではなく色のブロックを表示し、軽量化する
  "diffEditor.ignoreTrimWhitespace": false, // 差分表示で行末等の空白差分も無視せず表示する

  "security.workspace.trust.untrustedFiles": "open", // ワークスペースの信頼確認なしでファイルを開けるようにする

  // ターミナルがデフォルトで1000行までしか保存しないための設定
  "terminal.integrated.scrollback": 10000,

  // デバッグ時のツールバーを固定表示するための設定
  "window.commandCenter": true,
  "debug.toolBarLocation": "commandCenter",

  "zenMode.hideLineNumbers": false, // 行番号は隠さない

  "extensions.ignoreRecommendations": true, // 拡張機能のおすすめ通知を表示しない

  // Git
  "git.openRepositoryInParentFolders": "never",
  "git.ignoreLegacyWarning": true,

  // translate
  "vscodeGoogleTranslate.preferredLanguage": "Japanese",

  "editor.rulers": [100], // 100文字目に縦のガイド線を表示する

  // ターミナルパネルのデフォルト表示位置
  "workbench.panel.defaultLocation": "right",

  "[jsonc]": {
    "editor.formatOnSave": true, // 保存時に自動フォーマット
    "editor.formatOnType": true, // 入力時に自動フォーマット
    "editor.codeActionsOnSave": {
      "source.fixAll": "explicit", // 保存時にlintの自動修正を行う
      "source.organizeImports": "explicit" // 保存時にimportを整理する
    },
    "editor.defaultFormatter": "esbenp.prettier-vscode" // フォーマッタにPrettierを指定
  },

  "javascript.preferGoToSourceDefinition": true, // 右クリックで「Go to Source Definition」を選ばなくても、デフォルトのジャンプ動作がソース定義になる
  "typescript.preferGoToSourceDefinition": true, // TypeScriptでも同様にデフォルトのジャンプ動作をソース定義にする
  "[javascript][typescript][typescriptreact]": {
    // 保存時・タイプ時に自動的にフォーマットされる
    "editor.formatOnSave": true,
    "editor.formatOnType": true,
    // 拡張のPrettierをインストールしている必要がある
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.codeActionsOnSave": {
      "source.fixAll": "explicit", // 保存時にlintの自動修正を行う
      "source.fixAll.eslint": "explicit", // 保存時にESLintの自動修正を行う
      "source.organizeImports": "explicit", // 保存時にimportを整理する
      "source.addMissingImports": "explicit" // 保存時に不足しているimportを自動追加する
    }
  }

  // SQL
  // 下記を指定してみたが、フォーマットされなかったため、エクステンション無しではできないと考えられる(IME)。
  // "[sql]": {
  //     "editor.formatOnSave": true,
  //     "editor.codeActionsOnSave": {
  //         "source.fixAll": "explicit",
  //     },
  // },
}
```

## フォントサイズの変更
### エディタフォントのみのサイズ変更(v1.24〜)
* (参考) https://qiita.com/12345/items/64f4372fbca041e949d0

## 配色・テーマ
### 色を変える
* (参考) https://qiita.com/deren2525/items/6bc099ae8c05e3076055

### テーマカラーの変更
* コマンドパレットで「THEME」と入力し、「〜 Color Theme」を選んでから好きなカラーを選ぶ。

## ログファイル
```
/Users/***/Library/Application Support/Code/logs/
```

## preferences setting
* コマンドパレットで「preferences settings json」と検索すると、以下の設定を確認できる。
    * default settings
        * すべてのデフォルトの設定（読み取り専用）。
        * 拡張機能を追加すると、その拡張機能の設定も自動的にここへ追加される(?)。
    * user settings
    * workspace settings
        * `.vscode/settings.json`が開かれる。
