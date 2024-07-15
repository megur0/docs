[TOP(About this memo))](../README.md) > [一覧](./README.md) >


# WIP: 随時更新
* このメモは執筆中のため随時更新 

# 公式ドキュメント
* https://docs.flame-engine.org/latest/flame/flame.html

# GameWidget
* https://docs.flame-engine.org/latest/flame/game_widget.html
* GameWidgetはウィジェットツリー上のどの階層にも配置ができる。
* GameWidgetは可能な限り大きくなろうとする。
* GameWidgetはCanvas内のコンテンツをクリップしないため、範囲外にも描画を行う。
    * これを許容しない場合はFlutter標準のClipRectウィジェットで囲めば良い。
* ウィジェット内のbuildで、GameWidgetを生成するとそのウィジェットがリビルドされる度にGameWidgetもリビルドされる。
    * これを回避するためにはGameWidget.controlledを利用するか、ウィジェットの外で生成したGameWidgetを利用する
    * https://docs.flame-engine.org/latest/flame/game.html#flamegame