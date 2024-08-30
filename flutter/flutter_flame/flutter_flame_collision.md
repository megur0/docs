[TOP(About this memo))](../../README.md) >  [Flutter](../README.md) > [一覧](./README.md) >

# 衝突検出
* https://docs.flame-engine.org/latest/flame/collision_detection.html
* Forge2Dとの比較
    * https://docs.flame-engine.org/latest/flame/game.html
* HasCollisionDetection
    * with しておくと、配下のツリー上のコンポーネントに追加されたShapeHitboxが自動的に衝突を検知するように成る
* CollisionCallbacks
    * onCollisionやonCollisionEndによって衝突時の処理を記述する
* Hitbox
    * Componentにaddすることで、衝突を検知するように成る。
    * Hitbox同士のみ衝突する。
    * Hitboxは最も近い先祖のHasCollisionDetectionにのみ紐づく。(複数の先祖にHasCollisionDetectionが存在しても紐づくのは１つのみとなる。)
* ShapeHitbox
    * sizeが指定されない場合は親を可能な限り埋めるようなサイズとなる
    * PolygonHitbox, RectangleHitbox, CircleHitbox, 
* 以下はサンプルコード
    * スプライトは画面を一定速度で移動する
    * ScreenHitbox(Viewportの大きさ)に衝突すると方向を変える
    * PolygonHitbox(岩の周辺)に衝突すると岩のエフェクト
```dart
import 'package:flame/collisions.dart';
import 'package:flame/components.dart';
import 'package:flame/effects.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flame/palette.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(
    SafeArea(child: GameWidget(game: Example())),
  );
}

class Example extends FlameGame with HasCollisionDetection {
  Example() {
    debugMode = true;
  }

  static final _borderPaint = Paint()
    ..style = PaintingStyle.stroke
    ..strokeWidth = 2
    ..color = BasicPalette.red.color;

  static final TextPaint textRenderer = TextPaint(
    style: const TextStyle(color: Colors.white70, fontSize: 12),
  );

  @override
  Future<void> onLoad() async {
    add(ScreenHitbox());
    add(Ember(position: Vector2(size.x / 2, size.y / 2)));
    add(Rock(Vector2(size.x / 2, size.y / 2)));

    final text = TextComponent(
      textRenderer: textRenderer,
      position: Vector2(0, 0),
      anchor: Anchor.topLeft,
    );
    text.text = "viewport: $size";
    add(text);
  }

  @override
  void render(Canvas canvas) {
    canvas.drawRect(size.toRect(), _borderPaint);
    super.render(canvas);
  }
}

class Ember extends SpriteAnimationComponent
    with HasGameReference, CollisionCallbacks {
  Ember({super.position, super.key})
      : super(
          priority: 2,
          size: Vector2.all(50),
          anchor: Anchor.center,
        );
  late final maxPosition =
      Vector2(game.size.x - size.x / 2, game.size.y - size.y / 2);
  late final minPosition = Vector2(size.x / 2, size.y / 2);
  final Vector2 velocity = Vector2(10, 10);

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
  void onCollisionStart(
    Set<Vector2> intersectionPoints,
    PositionComponent other,
  ) {
    super.onCollisionStart(intersectionPoints, other);
    if (other is Rock) {
      other.add(
        ScaleEffect.to(
          Vector2.all(0.5),
          EffectController(duration: 0.2, alternate: true),
        ),
      );
    }

    if (other is ScreenHitbox) {
      final collisionPoint = intersectionPoints.first;
      // Left Side Collision
      if (collisionPoint.x == 0) {
        velocity.x = -velocity.x;
        velocity.y = velocity.y;
      }
      // Right Side Collision
      if (collisionPoint.x == game.size.x) {
        velocity.x = -velocity.x;
        velocity.y = velocity.y;
      }
      // Top Side Collision
      if (collisionPoint.y == 0) {
        velocity.x = velocity.x;
        velocity.y = -velocity.y;
      }
      // Bottom Side Collision
      if (collisionPoint.y == game.size.y) {
        velocity.x = velocity.x;
        velocity.y = -velocity.y;
      }

      angle = -velocity.angleToSigned(Vector2(1, 0));
    }
  }

  @override
  void update(double dt) {
    super.update(dt);
    final deltaPosition = velocity * (10 * dt);
    position.add(deltaPosition);
    position.clamp(minPosition, maxPosition);
  }
}

class Rock extends SpriteComponent with HasGameRef, TapCallbacks {
  Rock(Vector2 position)
      : super(
          position: position,
          size: Vector2.all(50),
          priority: 1,
          anchor: Anchor.center,
        );

  @override
  Future<void> onLoad() async {
    sprite = await game.loadSprite('nine-box.png');
    //add(RectangleHitbox());
    add(PolygonHitbox.relative(
      [
        Vector2(0, -5.0),
        Vector2(-6, 0),
        Vector2(0, 5),
        Vector2(1, 3),
        Vector2(1, -3),
      ],
      parentSize: size,
    ));
  }
}
```
