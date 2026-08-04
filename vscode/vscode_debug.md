---
title: "デバッグ機能 - VSCode"
updated: 2026-08-02
---

[TOP(About this memo))](../README.md) > [一覧(VSCode)](./README.md) > デバッグ機能

（題材はFlutter）

## 基本方針
* print文ではなく、なるべくブレークポイントを使う(IMO)。いちいちprintしなくても済むことが多い。
* ブレークポイントは都度外したり付けたり、次に進めたりできるため、かなり柔軟に扱える。
* ローカル変数やクラスの`this`の各フィールドも追えるほか、プログラムの動作そのものを追うことができる。
* SDKを調査したい場合
    * VSCode下部の「debug my code」を切り替えて「debug my code + packages + SDK」にすると、SDKなどにもブレークポイントを設定できる。

## エラー発生時のトレース
* 例外が発生した場合、自分でcatchしていてもいなくても、その例外の箇所で一旦VSCode側が処理を止めるようになっている。continueボタンを押すと処理が進む。
* step in / out / over、continueの使い分け
    * step into: 次の処理へ進む（中に入る）
    * step over: 次の処理へ進む（中には入らない）
    * step out: 現在のブロックを抜けた場所まで進む
    * continue: 次のブレークポイントまで進む

## conditional breakpoint (WIP)
* 右クリックで追加できる。
* フレームワークなどのブレークポイントが大量に呼ばれてしまう場合、条件を追加することで狙ったタイミングで確認できる。
    * 例: Expressionで `newWidget.runtimeType.toString() == 'Page2'`
* https://code.visualstudio.com/docs/editor/debugging#_advanced-breakpoint-topics

## log point
* 出力したい変数を`{}`で囲む。

## watch
* 見たい変数を監視できる。
* variableパネルなどで探さなくても、変数を選択して右クリックするだけですぐ見られるのが便利。
* ただし、privateな変数はエラーになるため、あまり有用ではないかもしれない(IMO)。
