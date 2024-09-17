[TOP(About this memo))](../../README.md) >  [Flutter](../README.md) > [一覧(Flame)](./README.md) > PositionComponent

# PositionComponent
* https://pub.dev/documentation/flame/latest/components/PositionComponent-class.html
    > A Component implementation that represents an object that can be freely moved around the screen, rotated, and scaled.
* Transform2Dクラス
    * position, scale, angleは内部ではこのクラスを利用している。
    * https://pub.dev/documentation/flame/latest/game/Transform2D-class.html
* ローカル座標(local coordinate)とグローバル座標(global coordinate)
    * グローバル座標はビューポートの左上を原点(0,0)とした座標
    * ローカル座標は相対的な座標であり、例えば親Aと子Bに対して、子Bにおけるローカル座標は、原点を親Aの左上とした座標となる。
* position
    * 親に対する相対位置（ローカル座標）として指定する。
        * 親がFlameGameの場合はビューポートに対する相対位置（グローバル座標）と同じとなる。
    * コンポーネントは、ローカルの原点(親の左上)を基準としたpositionの場所にそのコンポーネントの「論理的な中心」が位置するように配置される。
        * この論理的な中心は、デフォルトが自身の左上だが、anchorプロパティによって変更できる。
* size
    * カメラのズームが1.0の際の大きさ。親とは関係のない値となる。
    * 指定がない場合はゼロのサイズとなる。
* scale
    * コンポーネントとその子コンポーネントのスケール
* angle
    * 親を基準とした角度
* nativeAngle
    * angleが0の時のコンポーネントの向き
    * TODO: 具体例
* anchor
    * anchorで指定した位置が、コンポーネントの論理的な「中心」となる。
    * postionは、このanchorで指定されたコンポーネントの中心が、どの座標に位置するかを表す
    * 注意点として、親や子のanchorの値に関わらず、子のpositionの基準となる原点(※)は親の左上となる。
        * local origin。ローカル原点。子から見た0,0の位置。
    ```dart
    // ~/.pub-cache/hosted/pub.dev/flame-1.18.0/lib/src/anchor.dart
    class Anchor {
        static const Anchor topLeft = Anchor(0.0, 0.0);
        static const Anchor topCenter = Anchor(0.5, 0.0);
        static const Anchor topRight = Anchor(1.0, 0.0);
        static const Anchor centerLeft = Anchor(0.0, 0.5);
        static const Anchor center = Anchor(0.5, 0.5);
        static const Anchor centerRight = Anchor(1.0, 0.5);
        static const Anchor bottomLeft = Anchor(0.0, 1.0);
        static const Anchor bottomCenter = Anchor(0.5, 1.0);
        static const Anchor bottomRight = Anchor(1.0, 1.0);

        final double x;
        final double y;
        //...
    }
    ```
    ```dart
    // 公式ドキュメントから引用
    final comp = PositionComponent(
        size: Vector2.all(20),
        anchor: Anchor.center,
    );

    // Returns (0,0)
    // 生成時にアンカーをAnchor.centerとしているため、図形の真ん中が親のローカル原点の位置(この場合は親はGameサイズと同じ場合として(0,0))になる。
    final p1 = component.position;

    // Returns (10, 10)
    // 図形の真ん中が(0,0)のため、図形の右下の座標は(10, 10)となる
    final p2 = component.positionOfAnchor(Anchor.bottomRight);
    ```

# ShapeComponents
* PolygonComponent, RectangleComponent, CircleComponent
```dart
@override
Future<void> onLoad() async {
    add(RectangleComponent(size: Vector2(10, 10)));
}
```

# ParallaxComponent
* https://github.com/flame-engine/flame/tree/main/examples/lib/stories/parallax
* 複数のイメージを重ねて異なるスピードで動かすことができる
* ParallaxComponentにparallaxを指定して利用する
    ```dart
    // 例1
    final parallax = Parallax(
        //...
    );
    final parallaxComponent = ParallaxComponent(parallax: parallax);
    add(parallaxComponent);
    ```
    ```dart
    // 例2
    class MyParallaxComponent extends ParallaxComponent {
        @override
        Future<void> onLoad() async {
            parallax = await game.loadParallax(
                [
                    ParallaxImageData('parallax/bg.png'),
                    //...
                ],
            );
        }
    }
    ```


# イメージ、Sprite、Animation
* https://docs.flame-engine.org/latest/flame/rendering/images.html
* Loading images
    * Imagesクラス
        * 内部でdart.ui.Imageのキャッシュを保持する。
            ```dart
            // flame-1.19.0/lib/src/cache/images.dart
            final Map<String, _ImageAsset> _assets = {};
            ```
        * load
            * 下記のようにキャッシュに存在すればキャッシュから返し、なければロードする。
            ```dart
            Future<Image> load(String fileName, {String? key}) {
                return (_assets[key ?? fileName] ??=
                        _ImageAsset.future(_fetchToMemory(fileName)))
                    .retrieveAsync();
            }
            //...
            Future<Image> _fetchToMemory(String name) async {
                final data = await bundle.load('$_prefix$name');
                final bytes = Uint8List.view(data.buffer);
                return decodeImageFromList(bytes);
            }
            ```
        * clearCache
        * fromCache
    * static Flame.images
        * ゲーム全体で共有するImagesオブジェクト
    * static Game.images
        * Flame.imagesにアクセスする。
* イメージのパス
    * デフォルトでプレフィクスが"assets/images/"となっている。
    * 下記のように変更可能
    ```dart
    Flame.images.prefix = "assets/";
    ```
* ネットワーク画像
    * https://docs.flame-engine.org/latest/flame/rendering/images.html#loading-images-over-the-network
    * Flameでは特定のhttpパッケージに依存しないために、直接ネットワーク画像を取得するAPIは提供していない
    * Flameで扱うイメージはdart.ui.Imageのため、いずれのパッケージでもバイト列から変換すれば問題ない。
    * キャッシュを利用したい場合はFlame.images.fetchOrGenerateを利用するとよいだろう。
        ```dart
        import 'package:http/http.dart';
        //...
        image = await Flame.images
            .fetchOrGenerate("キャッシュ用の名前", () async {
            final response = await get(Uri.parse("https://docs.flutter.dev/assets/images/dash/Dash.png"));
            return decodeImageFromList(response.bodyBytes);
        });
        ```
## SpriteComponent
* https://pub.dev/documentation/flame/latest/components/SpriteComponent-class.html
* https://docs.flame-engine.org/latest/flame/components.html#spritecomponent
* Sprite
    * Canvas上へレンダリングできるImageの領域
    * static Game.loadSprite
        * 内部ではSprite.loadを呼んでいる
    * static Sprite.load
        * Flame.images.load から awaitして Image を取得して、Spriteにセットして返す。
```dart
@override
Future<void> onLoad() async {
    add(SpriteComponent(sprite: await game.loadSprite('xxx/yyy.png'), anchor: Anchor.center, size: Vector2.all(128.0), position: Vector2(10, 20), angle:pi/2));
}
```
* size
    * sizeを設定しない場合はイメージ本体のサイズ(Sprite.srcSize)となる。
## SpriteAnimationComponent
* https://pub.dev/documentation/flame/latest/components/SpriteAnimationComponent-class.html
* SpriteAnimationComponent.animationにSpriteAnimationを設定することで、そのアニメーションにしたがって処理される
* SpriteAnimation
    * factory SpriteAnimation.spriteList
        * List<Sprite>から作成
    * factory SpriteAnimation.fromFrameData
        * List<Sprite>から作成
        * それぞれのSpriteにstepTimesを指定できる
    * factory SpriteAnimation.fromFrameData
        * 以下から作成
        * dart:ui.Image: 各フレームの画像全てを保有しているイメージ
        * SpriteAnimationData: イメージファイルのどこにスプライトが含まれているかを表す情報
    * 他省略
* SpriteAnimationComponent
    * 無名コンストラクタ
        * SpriteAnimationを指定
    * SpriteAnimationComponent.fromFrameData
        * dart:ui.Image: 各フレームの画像全てを保有しているイメージ
        * SpriteAnimationData: イメージファイルのどこにスプライトが含まれているかを表す情報
* SpriteAnimationTicker
    * SpriteAnimationComponent の内部では、SpriteAnimationTickerを作成している。
    * SpriteAnimationComponent.update
        * SpriteAnimationTicker.updateによってSpriteAnimationTicker.currentIndexが更新される
    * SpriteAnimationComponent.render
        * spriteAnimation.frames[currentIndex].sprite で現在のSpriteを取り出して、Sprite.renderを実行
    * なお、SpriteAnimation.createTicker()によってSpriteAnimationTickerを取得することもできる
        * Tickerの終了を待ったり、開始時や終了時の処理を記述できる。
## TODO 
* SpriteBatch
* SpriteSheet  
* ImageComposition
* Animation  

# TextComponent
* TextComponentは生成時にsizeが自動で計算されて設定される。
* TextPaintクラスはTextPainterクラスへ変換できる。事前にサイズを取得したい場合は利用できるだろう。
```dart
final p = TextPaint(
      style: const TextStyle(
        fontSize: 32,
        color: Colors.white,
      ),
    );
final pSize = p.toTextPainter('Test').size;
print(pSize);
TextComponent(text: 'Test', textRenderer: p);
```


# ButtonComponent
* ButtonComponentのsizeはコンストラクタのみ引数のbuttonのサイズとして評価される点に注意
* 以下はコンストラクタでbuttonを指定した場合とonLoad内で設定した場合の比較
```dart
import 'dart:async';

import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flame/input.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(
    SafeArea(child: GameWidget(game: TextExample())),
  );
}

class TextExample extends FlameGame {
  TextExample() {
    debugMode = true;
  }
  static final textPaint = TextPaint(
    style: const TextStyle(
      fontSize: 32,
      color: Colors.white,
    ),
  );

  @override
  Future<void> onLoad() async {
    add(Child(position: Vector2(size.x / 2, size.y / 2)));

    // buttonを引数として渡さない場合
    add(_MyButton(
      onPressed: () {
        removeAll(children);
      },
    ));

    // buttonを引数として渡す場合
    // こちらはsizeがコンストラクタ(Initializer)にて設定される
    add(ButtonComponent(
      position: Vector2(size.x / 2, 0),
      button:
          TextComponent(text: 'Remove2', textRenderer: TextExample.textPaint),
      buttonDown: TextComponent(
          text: 'Remove2',
          textRenderer: TextExample.textPaint
              .copyWith((style) => style.copyWith(color: Colors.black))),
      onPressed: () {
        removeAll(children);
      },
    ));
  }
}

class _MyButton extends ButtonComponent {
  _MyButton({super.onPressed});

  @override
  FutureOr<void> onLoad() {
    button =
        TextComponent(text: 'Remove1', textRenderer: TextExample.textPaint);
    buttonDown = TextComponent(
        text: 'Remove1',
        textRenderer: TextExample.textPaint
            .copyWith((style) => style.copyWith(color: Colors.black)));

    // sizeを設定しないとsizeがゼロの状態のためボタンが反応しない
    //size = button!.size;
  }
}

class Child extends SpriteComponent {
  Child({super.position, super.key})
      : super(
          size: Vector2.all(100),
          anchor: Anchor.center,
        );

  @override
  Future<void> onLoad() async {
    sprite = await Sprite.load('flame.png');
  }
}

```

# その他のPositionComponent派生クラス
* SpriteAnimationGroupComponent
* SpriteGroupComponent
* SpawnComponent
    * 親の内部に他のコンポーネントを生成する非ビジュアル コンポーネント
* IsometricTileMapComponent
* NineTileBoxComponent
* CustomPainterComponent
* ClipComponent
