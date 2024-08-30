[TOP(About this memo))](../../README.md) >  [Flutter](../README.md) > [一覧](./README.md) >



# CameraComponent と World
* https://docs.flame-engine.org/latest/flame/camera_component.html
* https://github.com/flame-engine/flame/blob/main/examples/lib/stories/camera_and_viewport/coordinate_systems_example.dart
## World
* https://pub.dev/documentation/flame/latest/camera/World-class.html
    > このコンポーネントの主な機能は、通常のレンダリングを無効にし、 CameraComponentを通じてのみレンダリングできるようにすることです。  
    > 更新は、通常どおりワールド ツリーを通じて進行します。
* https://docs.flame-engine.org/latest/flame/camera_component.html#world
* Component派生クラスで、ルートのコンポーネントとなる。
* Worldが他のコンポーネントと異なる点は、1 つ以上のCameraComponentによってレンダリングされる点になる。
* 多くの場合、Worldクラスを拡張して独自のワールドクラスを作成して、そこにロジックを実装(onLoadなど)を実装する。
* Gameは複数のWorldを持つこともできる。カメラのターゲットとなるワールドを切り替えて、レンダリングされるワールドを制御することができる
## CameraComponent
* 概念としては、CameraComponentはWorldを「見る」役割となる。
* 生成時にWorldが引数で必要だが、後で別のものに置き換えることもできる
* ViewportとViewfinderというコンポーネントを内部に持つ
    * CameraComponent.viewportはWorldを見る際の「窓枠」となる
    * CameraComponent.viewfinderはWorldを見る際の「カメラの位置やズーム、角度、etc」となる
* サンプル: カメラの位置をキーボードで動かすことでレンダリングされる位置が変わる。
    * なお、Canvas内のコンテンツはクリップされないため、viewportの枠をはみ出ても描画される。
    * https://github.com/flame-engine/flame/blob/main/examples/lib/stories/camera_and_viewport/coordinate_systems_example.dart
    * https://examples.flame-engine.org/#/Camera_%26_Viewport_Coordinate_Systems
* CameraComponentはBackdropを内部で持つ
* CameraComponent.canSeeで対象のコンポーネントがカメラから見えるかをチェックできる
* CameraComponent.withFixedResolution
    * 画面の中央にx, yのアスペクト比のビューポートを持つカメラを作成する。アスペクト比を保ちながら可能な限りスペースを利用する
* CameraComponent.setBounds
* CameraComponent.follow
    * サンプル
    * https://github.com/flame-engine/flame/blob/main/examples/lib/stories/camera_and_viewport/follow_component_example.dart
* CameraComponent.visibleWorldRect
    * viewfinder.visibleWorldRectを返す
* カメラのコントロール
    * https://docs.flame-engine.org/latest/flame/camera_component.html#camera-controls
## Viewport
* Viewportに子コンポーネントを追加すると全面に静的なHUD(ヘッドアップディスプレイ)として表示することができる。
* Viewport.size
* Viewport.anchor
* Viewport.position
* Viewportの種類
    * MaxViewport(デフォルト)
        * gameで許可された最大サイズまで大きくなる。
    * その他、FixedResolutionViewport, FixedSizeViewport, FixedAspectRatioViewport, CircularViewport
## ViewFinder
* ViewFinderに子コンポーネントを追加すると、Worldの前に表示され、Viewportの後ろに表示され、Worldと同じトランスフォームが適用される。
* ViewFinder.angle
* ViewFinder.anchor
* ViewFinder.zoom
* ViewFinder.position
## Backdrop
* ワールドの下に静的なコンポーネントを置くことができる。(例えばParallaxComponentを置くなど)
## サンプルコード
* 以下はドラッグでカメラの位置やズームを操作するサンプルとなる
    * カメラの初期位置は0,0で、ワールド内のテキストも0,0のため、画面上は中心に配置される。（カメラから見て対象は正面に存在するため）
    * ズームに関してはiOSで試したが想定通りの動作にはできなかった
```dart
import 'dart:async';

import 'package:flame/components.dart';
import 'package:flame/events.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(
    SafeArea(child: GameWidget(game: Example2())),
  );
}

class Example2 extends FlameGame with DragCallbacks, ScaleDetector {
  Example2()
      : super(
          world: _MyWorld(),
        ) {
    debugMode = true;
  }

  final cameraPosition = Vector2.zero();
  final scale = 1.0;

  late final TextComponent text;

  @override
  FutureOr<void> onLoad() {
    add(text = TextComponent(text: "", position: Vector2(0, 0)));
  }

  @override
  void update(double dt) {
    text.text =
        "camera pos: ${camera.viewfinder.position.x.ceil()}, ${camera.viewfinder.position.y.ceil()}\ncamera zoom: ${camera.viewfinder.zoom}";
    super.update(dt);
  }

  @override
  void onDragUpdate(DragUpdateEvent event) {
    cameraPosition.add(event.canvasDelta);
    camera.viewfinder.position = cameraPosition;
  }

  late double startZoom;

  @override
  void onScaleStart(ScaleStartInfo info) {
    print("onScaleStart: ${info.raw}");
    startZoom = camera.viewfinder.zoom;
  }

  @override
  void onScaleUpdate(ScaleUpdateInfo info) {
    // iOSで試したところ、scaleがほとんど0.0となってしまう。
    // 繰り返し操作をしているとscaleが取得できて想定通りズームできるときもある。
    // 現状は解決手段が分からない。
    print("onScaleUpdate: ${info.scale.global}");
    if (info.scale.global.y == 0.0) return;
    final currentScale = info.scale.global.y.clamp(0.5, 3.0);
    camera.viewfinder.zoom = startZoom * currentScale;
  }
}

class _MyWorld extends World with HasGameReference {
  @override
  Future<void> onLoad() async {
    add(TextComponent(text: "on World", position: Vector2(0, 0)));
  }
}

```