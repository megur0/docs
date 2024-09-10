[TOP(About this memo))](../../README.md) >  [Flutter](../README.md) > [一覧(Flame)](./README.md) > 各種ユーティリティ（未整理）


# mixin XXX on Component
* TapCallbacks
    ```dart
    class Rock extends SpriteComponent with TapCallbacks {
        //...
        @override
        void onTapDown(_) {
                add(
                ScaleEffect.to(
                    Vector2.all(scale.x >= 2.0 ? 1 : 2),
                    EffectController(duration: 0.3),
                ),
            );
        }
    }
    ```
* DoubleTapCallbacks
* CollisionCallbacks
* HoverCallbacks
* HasGameReference, HasGameRef
    * HasGameReferenceはHasGameRefの新しいバージョン
* HasWorldReference
* HasVisivility
    * isVisible を falseにするとレンダリングされない。
    * removeはアンマウントされライフサイクルメソッドが呼ばれるが、こちらは単に非表示にするような手段としての利用が可能となる。
    * https://docs.flame-engine.org/latest/flame/components.html#visibility-of-components
    ```dart
    // 実装
    mixin HasVisibility on Component {
        bool isVisible = true;

        @override
        void renderTree(Canvas canvas) {
            if (isVisible) {
            super.renderTree(canvas);
            }
        }
    }
    ```
* HasTimeScale
* HasCollisionDetection
* HasKeyboardHandlerComponents
* HasDecorator
* KeyboardHandler
* 他

# mixin XXX on Game
* TapDetector
* DoubleTapDetector
* LongPressDetector
* ScaleDetector
* ScrollDetector
* MultiTouchTapDetector
* MultiTouchDragDetector
* KeyboardEvents
* 他

# エフェクト
* https://docs.flame-engine.org/latest/flame/effects.html
* Flameには内蔵のエフェクトが用意されている。
* 独自に作成することもできる。
* EffectController
    * https://docs.flame-engine.org/latest/flame/effects.html#effect-controllers
    * エフェクトが時間の経過とともにどのように変化するかを記述するオブジェクト
    * durationは 秒数を表す(Durationクラスではなくdoubleを使っているようだ)
```dart
@override
  void onTapDown(TapDownEvent event) {
    player.add(
      MoveToEffect(
        event.localPosition,
        EffectController(
          duration: 1.0,
        ),
      ),
    );
  }
```

# 色とパレット
* flameが提供するBasicPalette、PaletteEntry は dart.uiのColor, Paintをラップしているユーティリティクラス
    * https://docs.flame-engine.org/latest/flame/rendering/palette.html


# デコレータ
* https://docs.flame-engine.org/latest/flame/rendering/decorators.html
* HasDecoratorをwithすることで利用できる
* PositionComponentは既にdecoratorを保持しており、既に内部で利用しているため、PositionComponent.decorator.addLast によってデコレータを加える。
* 以下はサンプルコード
```dart
import 'dart:async';
import 'dart:math';

import 'package:flame/components.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flame/rendering.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(
    SafeArea(child: GameWidget(game: DecoratorExample())),
  );
}

class DecoratorExample extends FlameGame with DragCallbacks, ScaleDetector {
  DecoratorExample() {
    debugMode = true;
  }

  @override
  FutureOr<void> onLoad() {
    final p = Child(position: Vector2(size.x / 2, size.y / 2));
    add(p);
  }
}

class Child extends SpriteComponent with TapCallbacks {
  Child({
    super.position,
    super.key,
  }) : super(
          size: Vector2.all(100),
          anchor: Anchor.center,
        );

  final decoratorList = <Decorator>[
    PaintDecorator.blur(3.0),
    PaintDecorator.grayscale(opacity: 0.5),
    PaintDecorator.tint(const Color(0xAAFF0000)),
    Rotate3DDecorator(
      center: Vector2(50, 50),
      angleX: pi / 4,
      perspective: 0.01,
    ),
    Shadow3DDecorator(
      base: Vector2(100, 150),
      angle: -1.4,
      xShift: 200,
      yScale: 1.5,
      opacity: 0.5,
      blur: 1.5,
    ),
  ];

  int index = 0;

  @override
  Future<void> onLoad() async {
    sprite = await Sprite.load('flame.png');
  }

  @override
  void onTapUp(TapUpEvent event) {
    if (index > 0) decorator.removeLast();
    decorator.addLast(decoratorList[(index++) % decoratorList.length]);
  }
}
```

# オーバーレイ
* https://docs.flame-engine.org/latest/flame/overlays.html
* GameWidget.overlayBuilderMapでFlutterのウィジェットをMap形式で渡すことができる。
* Game.overlays.removeやaddでMapのキーを渡すことでComponentのように追加、削除ができる。


# AlignComponent
```dart
void main() {
  runApp(
    SafeArea(child: GameWidget(game: DecoratorExample())),
  );
}

class DecoratorExample extends FlameGame with DragCallbacks, ScaleDetector {
  DecoratorExample() {
    debugMode = true;
  }

  @override
  FutureOr<void> onLoad() {
    for (final anchor in Anchor.values) {
      add(AlignComponent(
          child: TextComponent(text: anchor.toString()), alignment: anchor));
    }
  }
}
```

# Notifiler
* ChangeNotifierの派生クラス
* ComponentsNotifier, ComponentsNotifierBuilder
* https://github.com/flame-engine/flame/blob/main/examples/lib/stories/components/components_notifier_example.dart

# (未読)パーティクル
* https://github.com/flame-engine/flame/blob/main/examples/lib/stories/rendering/particles_example.dart
* https://docs.flame-engine.org/latest/flame/rendering/particles.html

# (未読)ルーティング
* https://docs.flame-engine.org/latest/flame/router.html

# (未読)レイヤーとスナップショット
* https://docs.flame-engine.org/latest/flame/rendering/layers.html

# (未読)デバッグ、Util、ウィジェット
* https://docs.flame-engine.org/latest/flame/other/other.html
