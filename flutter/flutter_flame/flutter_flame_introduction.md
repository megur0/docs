[TOP(About this memo))](../../README.md) >  [Flutter](../README.md) > [一覧(Flame)](./README.md) > 導入

# 公式ドキュメント
* https://docs.flame-engine.org/latest/flame/flame.html

# Flutter公式のガイド
* https://docs.flutter.dev/resources/games-toolkit
* https://flutter.dev/games

# API
* https://pub.dev/documentation/flame/latest/

# サンプルコード
* 目的別のサンプルコード
    * https://github.com/flame-engine/flame/tree/main/examples
    * ドキュメントの各コンテンツについて、動作するサンプルコードも多く含まれている。
* シンプルな完全な構成例
    * https://github.com/flame-engine/flame/blob/main/packages/flame/example/lib/main.dart
* 各サンプルゲーム
    * https://github.com/flame-engine/flame/tree/main/examples/games
    * https://github.com/marismar/ember-quest
    * ファイルの構成、コンポーネントの構成の参考となる
* Flutter公式
    * https://flutter.dev/games
    * カードゲームや横スクロールアクションや基本テンプレートなどの素材がある
    * 「Super Dash Demo Game」というストア上にリリースしているアプリのコードも提供されている

## サンプルコードのデモ
* https://examples.flame-engine.org/
* 右のメニューからソースコードへジャンプすることもできる
## サンプルコードの中の一つを手元で動かす
* Flutterプロジェクトを作成する
* flutter pub add flame
    * サンプルによって他にも必要なパッケージがある場合はそちらも加える
* assets, commonsフォルダの配置
    * flameリポジトリをダウンロード
    * examples/assetsをディレクトリごと手元のプロジェクトへコピーする
    * examples/lib/commonsをディレクトリごと手元のプロジェクトへコピーする
* assetsをpubspec.ymlへコピー
    ```
    assets:
        - assets/images/animations/
        - assets/images/
        - assets/images/tile_maps/
        - assets/images/layers/
        - assets/images/parallax/
        - assets/images/parallax/
        - assets/images/rogue_shooter/
        - assets/spine/
        - assets/svgs/
        - assets/tiles/
        - assets/audio/music/
        - assets/audio/sfx/
        - assets/yarn/
    ```
* サンプルコードをコピー
    * サンプルコードはFlameGameの派生クラスで作成されているため、GameWidgetに渡すことで実行できる
    * ただし、コードによってはGameWidgetではないウィジェットや、引数を渡す必要があるコードもあるため注意。
    ```dart
    void main() {
    runApp(
        GameWidget(
            game: サンプルコード内のFlameGameの派生クラス(),
        ),
    );
    }
    /*
    対象のサンプルコードをコピー
    */
    ```
* 実行

# 作成したゲームをGithub Pagesで公開する
* https://docs.flame-engine.org/latest/flame/platforms.html#deploy-your-game-to-github-pages

# ファイル構成
* https://docs.flame-engine.org/latest/flame/structure.html


# (注意)名前の衝突について
* 例えば下記のクラスはFlutter Flamework のウィジェットやdartのクラスと衝突する。
    * flame-1.19.0/lib/src/timer.dartのTimerクラス
    * forge2d/src/common/transform.dartのTransFormクラス
* 下記のように名前付きインポートを併用すれば良い。
```dart
import 'dart:async';
import 'dart:async' as dart;
import 'package:flame_forge2d/flame_forge2d.dart' as forge2d;
import 'package:flame_forge2d/flame_forge2d.dart';
import 'package:flutter/material.dart' as material;
import 'package:flutter/material.dart';
// あるいは下記のように特定のクラスを隠す方法もある
// import 'package:flame_forge2d/flame_forge2d.dart' hide Transform;

//...
dart.Timer bubbleTimer;

//...
@override
Widget build(BuildContext context) {
      //...
      child: material.Transform.rotate(
        //...
      );
}
```
