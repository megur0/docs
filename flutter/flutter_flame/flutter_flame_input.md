[TOP(About this memo))](../../README.md) >  [Flutter](../README.md) > [一覧(Flame)](./README.md) > 入力

# 入力系
* https://docs.flame-engine.org/latest/flame/inputs/inputs.html
* 以下はサンプルコード
    * ドラッグによってスプライトを動かすことができる
    * スプライトはViewport内でのみ動かすことができる
    * 画面を押下するとスプライトは削除される
```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flame/palette.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(
    SafeArea(child: GameWidget(game: Example())),
  );
}

class Example extends FlameGame with HasCollisionDetection, TapCallbacks {
  Example() {
    debugMode = true;
  }

  static final _borderPaint = Paint()
    ..style = PaintingStyle.stroke
    ..strokeWidth = 2
    ..color = BasicPalette.red.color;

  late Ember ember;

  @override
  Future<void> onLoad() async {
    add(ScreenHitbox());
    add(ember = Ember(position: Vector2(size.x / 2, size.y / 2)));
  }

  @override
  void onTapUp(TapUpEvent event) {
    remove(ember);
    add(ember = Ember(position: Vector2(event.canvasPosition.x, event.canvasPosition.y)));
  }

  @override
  void render(Canvas canvas) {
    canvas.drawRect(size.toRect(), _borderPaint);
    super.render(canvas);
  }
}

class Ember extends SpriteAnimationComponent
    with HasGameReference, CollisionCallbacks, DragCallbacks {
  Ember({super.position, super.key})
      : super(
          priority: 2,
          size: Vector2.all(50),
          anchor: Anchor.center,
        );
  late final maxPosition =
      Vector2(game.size.x - size.x / 2, game.size.y - size.y / 2);
  late final minPosition = Vector2(size.x / 2, size.y / 2);

  @override
  Future<void> onLoad() async {
    animation = await game.loadSpriteAnimation(
      'animations/ember.png',
      SpriteAnimationData.sequenced(
        amount: 3,
        textureSize: Vector2.all(16),
        stepTime: 0.15,
      ),
    );
    add(CircleHitbox());
  }

  @override
  void onDragUpdate(DragUpdateEvent event) {
    position += event.canvasDelta; // event.localDeltaは angle の角度の影響を受ける
    position.clamp(minPosition, maxPosition);
  }
}
```
* Eventクラス
    * continuePropagationをtrueとすると、複数のコンポーネントに対してのイベントを許可する。(例えば重なっているコンポーネント)
        * https://github.com/flame-engine/flame/blob/main/examples/lib/stories/input/overlapping_tap_callbacks_example.dart
