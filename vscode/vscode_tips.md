---
title: "便利機能・Tips - VSCode"
updated: 2026-08-02
---

[TOP(About this memo))](../README.md) > [一覧(VSCode)](./README.md) > 便利機能・Tips


## Google翻訳機能
### 選択範囲を優先言語に翻訳
* 選択 → `shift + opt + T` → 言語選択（2回目以降は選択不要）
    * コード自体が書き換わるので注意。

### 任意の言語へ変換
* コマンドパレットから「Translate Selection」

### コメントを翻訳する際の手順
* コピーして別タブに貼り付ける。
* `//`を削除（`cmd + K` → `cmd + U`、もしくは`cmd + /`）
    * 問題点: untitledなファイルだと、コメント行が`//`だったり`<!-- -->`だったりして統一されないため、うまく動作しないことが多い。空白文字に置換するしかないかもしれない(IME)。
* 改行を削除（`ctrl + J`）
* 翻訳（`shift + opt + T`）
* (参考) https://1-notes.com/visual-studio-code-google-translate/
* 上記の手順をショートカット化できないか、今後検討したい(TODO)。

## 便利機能
* ピン留め
* スニペット
    * (参考) https://qiita.com/ShinyaOkazawa/items/bc86a876a16646c0002a
* Multi-root Workspace
    * https://code.visualstudio.com/docs/editor/multi-root-workspaces
