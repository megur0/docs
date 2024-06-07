- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)




# Ticker, TickerProvider
* https://api.flutter.dev/flutter/scheduler/Ticker-class.html
* https://api.flutter.dev/flutter/scheduler/TickerProvider-class.html
* Tickerは(アニメーション)フレーム毎にコールバックを実行する
    * SchedulerBindingによって駆動する。
* TickerProviderはTickerオブジェクトを提供する抽象クラス
    * TickerProviderStateMixin, SingleTickerProviderStateMixin, TestVSyncTickerProvider, StateMixin, WidgetTester が実装している。
    * 基本的には開発者はTickerProviderStateMixinやSingleTickerProviderStateMixinを利用してTickerを生成して、そのTickerにコールバックを実行させることが多いだろう。
* サンプルコード
    * 下記のようにTickerにコールバックを渡して値を毎フレーム更新することで、アニメーションを実現できる。
    ```
    import 'package:flutter/material.dart';
    import 'package:flutter/scheduler.dart';

    void main() => runApp(const MaterialApp(home: MyTickerTest()));

    class MyTickerTest extends StatefulWidget {
        const MyTickerTest({super.key});

        @override
        State<MyTickerTest> createState() => _MyTickerTestState();
    }

    double? _x;
    double? _y;

    class _MyTickerTestState extends State<MyTickerTest>
        with SingleTickerProviderStateMixin {
        late Ticker _ticker;

        @override
        void initState() {
            super.initState();
            _ticker = createTicker((elapsed) {
            final double elapsedInSeconds =
                elapsed.inMicroseconds.toDouble() / Duration.microsecondsPerSecond;
            if (elapsedInSeconds > 3) {
                _ticker.stop();
            }
            if (_x != null) {
                _x = _x! + 1.0;
                _y = _y! + 1.0;
            }

            setState(() {});
            });
            _ticker.start();
        }

        @override
        void dispose() {
            super.dispose();
            _ticker.dispose();
        }

        @override
        Widget build(BuildContext context) {
            return Scaffold(
            body: CustomPaint(
                foregroundPainter: MyPainter(),
                size: Size(MediaQuery.of(context).size.width,
                    MediaQuery.of(context).size.height),
            ),
            );
        }
    }

    class MyPainter extends CustomPainter {
        MyPainter();

        @override
        void paint(Canvas canvas, Size size) {
            if (_x == null) {
            _x = size.width / 2;
            _y = size.height / 2;
            }
            canvas.drawCircle(Offset(_x!, _y!), 10.0, Paint()..color = Colors.blue);
        }

        @override
        bool shouldRepaint(CustomPainter oldDelegate) {
            return true;
        }
    }

    ```

# Animation
* https://api.flutter.dev/flutter/animation/Animation-class.html
    > An animation with a value of type T.
* 抽象クラスでListenableサブクラスとなる。
* 扱うことが多い主な具象クラス
    * AnimationController
    * CurvedAnimation
    * Tweenのメソッドによって生成されるオブジェクト(runtimeTypeはプライベートクラス)
## AnimationController(Animationサブクラス)
* https://api.flutter.dev/flutter/animation/AnimationController-class.html
* Animationを継承する
* Tickerのstart, stop, disposeなどの処理や渡すコールバックなどをラップしたクラス。TickerProviderを引数のvsyncで指定する。
* 下記のような処理・機能を持つ。
    * 内部で１つのvalue(double)を持っており、時間の経過(Tickerのコールバックが渡されるDuration)とともに値を毎フレーム更新する。
    * valueの値は上限、下限(lowerBound, upperBound)を設定できる。
         * デフォルト値は0.0, 1.0
         * valueの値は下限から条件の範囲にclampDouble関数によって丸められる
         * AnimationController.unboundedコンストラクタ ではdouble.infinity, double.negativeInfinityが設定される
    * valueの値はコンストラクタ生成時にを設定、setterで変更できる。
         * デフォルト値は下限値となる。
         * resetでは、下限値が設定される。
    * シミュレーション
        * 値の遷移はシミュレーションに基づいて設定される。
        * forwardやreverese等のメソッドではvalueは現在値もしくは指定値から上限値(下限値)へ向かって線形に更新される。
        * animateWithでは、Simulationクラスを指定してそれに基づいてvalueの更新を行う。
        * Tickerの終了条件(stop)は、Simulation.isDoneとなったタイミングとなる。
            * fowardやreverseは指定されたdurationの時間で目的値へ到達するようにSimulationが生成される 。(durationが指定されていない場合はエラーとなる)
    * Curveクラスを指定して線形以外の遷移を指定できる。(animateTo, animateBack)
        * https://api.flutter.dev/flutter/animation/Curves-class.html
    * Listenableを実装
        * リスナーとしてコールバックを設定すると(addListener)、Tickerのコールバックが実行されるたび(毎フレーム)呼び出される。
    * その他、reset, repeat, fling, stopなどのAPIを提供
* 以下はデモのコードとなる
  ```
  import 'package:flutter/material.dart';
  import 'package:flutter/physics.dart';

  void main() {
    runApp(const MaterialApp(
        home: Scaffold(
      body: Center(child: AnimationControllerTest()),
    )));
  }

  class AnimationControllerTest extends StatefulWidget {
    const AnimationControllerTest({super.key});

    @override
    State<AnimationControllerTest> createState() =>
        _AnimationControllerTestState();
  }

  enum BoundType { bound1, bound100, unbound }

  class _AnimationControllerTestState extends State<AnimationControllerTest>
      with TickerProviderStateMixin {
    late AnimationController controller;
    BoundType boundType = BoundType.bound1;

    @override
    void initState() {
      super.initState();
      create(true);
    }

    create([bool initial = false]) {
      if (!initial) controller.dispose();

      switch (boundType) {
        case BoundType.bound1:
          controller = AnimationController(
              vsync: this, duration: const Duration(milliseconds: 1000));
        case BoundType.bound100:
          controller = AnimationController(
              upperBound: 100.0,
              lowerBound: -100,
              vsync: this,
              duration: const Duration(milliseconds: 1000));
        case BoundType.unbound:
          controller = AnimationController.unbounded(
              vsync: this, duration: const Duration(milliseconds: 1000));
      }

      controller.addListener(() {
        debugPrint(controller.value.toString());
        setState(() {});
      });
    }

    @override
    Widget build(BuildContext context) {
      return Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Text("duration: ${controller.duration}"),
          Text("value: ${controller.value.toString()}"),
          Text("velocity: ${controller.velocity.toString()}"),
          Text("status: ${controller.status.toString()}"),
          Text("animationBehavior: ${controller.animationBehavior}"),
          Text(
              "lastElapsedDuration: ${controller.lastElapsedDuration.toString()}"),
          Text(
              "lowerBound: ${controller.lowerBound}, upperBound: ${controller.upperBound}"),
          DropdownButton(
              value: boundType,
              items: BoundType.values
                  .map((e) => DropdownMenuItem(value: e, child: Text(e.name)))
                  .toList(),
              onChanged: (value) {
                boundType = value!;
                create();
                setState(() {});
              }),
          TextButton(
              onPressed: () {
                controller.forward();
              },
              child: const Text("foward")),
          TextButton(
              onPressed: () {
                controller.stop();
              },
              child: const Text("stop")),
          TextButton(
              onPressed: () {
                controller.reverse();
              },
              child: const Text("reverse")),
          TextButton(
              onPressed: () {
                controller.reset();
              },
              child: const Text("reset")),
          TextButton(
              onPressed: boundType == BoundType.bound1
                  ? () {
                      controller.repeat();
                    }
                  : null,
              child: const Text("repeat")),
          TextButton(
              onPressed: () {
                controller.fling();
              },
              child: const Text("fling")),
          TextButton(
              onPressed: () {
                controller.animateTo(1.0, curve: Curves.bounceIn);
              },
              child: const Text("animateTo(bounceIn)")),
          TextButton(
              onPressed: () {
                controller.animateWith(GravitySimulation(10.0, 0.0, 80.0, 0.0));
              },
              child: const Text("animateWith(GravitySimulation)")),
        ],
      );
    }
  }
  ```


# Animatable<T>
* https://api.flutter.dev/flutter/animation/Animatable-class.html
* Animation<double>からT型のオブジェクトを生成するクラス。
* 主な用途として、任意個数の目的別のAnimatableオブジェクトを使って、AnimationControllerから目的に応じたAnimationオブジェクトを生成(してvalueを利用)する
* 主な具象クラスは
    * Tween
        * https://api.flutter.dev/flutter/animation/Tween-class.html
    * CurveTween
        * https://api.flutter.dev/flutter/animation/CurveTween-class.html
    * TweenSequence
        * https://api.flutter.dev/flutter/animation/TweenSequence-class.html
        * 複数のTweenを基にしたシーケンスとなるAnimatable
* animateメソッド
    * Animationオブジェクトから別のAnimationオブジェクトを生成する
        * xxx = Tween.animate(AnimationController) によってAnimationクラスを生成
            * AnimationController.driveによって生成する事もできる。
            * 内部でAnimatable.animateを呼び出すため、同じ処理となる。
        * 引数としては渡すAnimationオブジェクトは、基本的にはAnimationControllerを利用する。
            * 基本的には0.0〜1.0の範囲が想定されることが多い。例えばCurveTweenは0.0〜1.0の範囲外となるとassert errorとなる。
    * 新しく生成されたAnimationオブジェクトのvalueは、元のAnimationのvalueの値を、目的別に変換した値となる。
        * 例えばTweenは、線形補間(linear interpolation)によって値を変換する。
            * 線形補間とは、ある２つの座標の間の任意の位置の値を算出する手法となる
* chainメソッド
    * 複数のTweenによる変換を組み合わせることができる。
    * 例えば AnimationControllerのvalue(0.0〜1.0) -> CurveTweenで値の変化にカーブをつける -> Tweenで変換 
* 以下はデモコード
```
import 'dart:math';

import 'package:flutter/material.dart';

class TweenTest extends StatefulWidget {
  const TweenTest({super.key});

  @override
  State<TweenTest> createState() => _TweenTestState();
}

class _TweenTestState extends State<TweenTest>
    with SingleTickerProviderStateMixin {
  final _scaleTween = Tween<double>(begin: 1.0, end: 0.01);
  final _rotateTween = Tween<double>(begin: 0, end: 360);
  final _duration = 3000;

  late AnimationController _animationController;
  late Animation _scaleAnimation;
  late Animation _rotateAnimation;
  late Animation _sequenseAnimation;

  @override
  void initState() {
    super.initState();

    _animationController = AnimationController(
      duration: Duration(milliseconds: _duration),
      vsync: this,
    );

    _scaleAnimation = _scaleTween.animate(_animationController)
      ..addListener(() {
        setState(() {});
      });

    _rotateAnimation = _rotateTween
        .chain(CurveTween(curve: Curves.easeIn))
        .animate(_animationController)
      ..addListener(() {
        setState(() {});
      });

    _sequenseAnimation = TweenSequence<double>(
      <TweenSequenceItem<double>>[
        TweenSequenceItem<double>(
          tween: Tween<double>(begin: 0.0, end: 300.0)
              .chain(CurveTween(curve: Curves.bounceOut)),
          weight: 40.0,
        ),
        TweenSequenceItem<double>(
          tween: ConstantTween<double>(300.0),
          weight: 40.0,
        ),
        TweenSequenceItem<double>(
          tween: Tween<double>(begin: 300.0, end: 0.0)
              .chain(CurveTween(curve: Curves.easeOut)),
          weight: 20.0,
        ),
      ],
    ).animate(_animationController);

    _animationController.repeat();
  }

  @override
  Widget build(BuildContext context) {
    return Stack(
      children: [
        Transform.translate(
          offset: Offset(0.0, _sequenseAnimation.value),
          child: const FlutterLogo(
            size: 100.0,
          ),
        ),
        Transform.translate(
          offset: Offset(
              0.0,
              MediaQuery.of(context).size.width *
                  0.5 *
                  _animationController.value),
          child: Transform.rotate(
            angle: pi / 180 * _rotateAnimation.value,
            child: Transform.scale(
              scale: _scaleAnimation.value,
              child: const SizedBox(
                height: 100.0,
                width: 100.0,
                child: ColoredBox(
                  color: Colors.black,
                ),
              ),
            ),
          ),
        )
      ],
    );
  }

  @override
  void dispose() {
    _animationController.dispose();
    super.dispose();
  }
}
```
## 内部処理
* valueをオーバーライドしてtransformで返しており、このtransfromをTweenが実装している。
    * AnimatableやTweenのサブクラスを定義する事で独自のTweenを定義できる。
```
abstract class Animatable<T> {
    //...
    T transform(double t);

    T evaluate(Animation<double> animation) => transform(animation.value);

    Animation<T> animate(Animation<double> parent) {
        return _AnimatedEvaluation<T>(parent, this);
    }

    Animatable<T> chain(Animatable<double> parent) {
        return _ChainedEvaluation<T>(parent, this);
    }
}
//...
class _AnimatedEvaluation<T> extends Animation<T> with AnimationWithParentMixin<double> {
    _AnimatedEvaluation(this.parent, this._evaluatable);
    // ...
    @override
    T get value => _evaluatable.evaluate(parent);
    // ...
}
```
* Tweenの場合は下記のような実装となっている。
```
class Tween<T extends Object?> extends Animatable<T> {
  //...
  @protected
  T lerp(double t) {
    return (begin as dynamic) + ((end as dynamic) - (begin as dynamic)) * t as T;
  }

  @override
  T transform(double t) {
    if (t == 0.0) {
      return begin as T;
    }
    if (t == 1.0) {
      return end as T;
    }
    return lerp(t);
  }
  //...
}
```


# アニメーションに関連するウィジェット
* https://docs.flutter.dev/ui/widgets/animation
## AnimatiedBuilder
* https://api.flutter.dev/flutter/widgets/AnimatedBuilder-class.html
* listenableを受け取って監視する。
* 実装はListenbleBuilderと同じもので、可読性を高めるために異なる名前のウィジェットとなっている。
* ListenableBuilderでも同じ目的は達成できるが、意図に従って使い分けをすると良いだろう。
## AnimatedWidget
* https://api.flutter.dev/flutter/widgets/AnimatedWidget-class.html
* 与えられたListenableの値が変わった際にリビルドされる。
* AnimatiedBuilderとの違いはこちらはabstract classで、拡張して利用する。
## コントローラやアニメーションオブジェクトを利用せずアニメーションを実現する
* 下記のウィジェットは、直接アニメーションのオブジェクトを操作せずに各目的を実現する。
* AnimatedContainer
* AnimatedAlign
* AnimatedCrossFade
* AnimatedDefaultTextStyle
* AnimatedOpacity
* AnimatedPhysicalModel
* AnimatedPositioned
* AnimatedSize
## アニメーションオブジェクトを直接渡すウィジェット
* トランジション系
    * Animation.valueを基に算出した値をTransform等のウィジェットに渡して画面に反映しても良いが、下記のウィジェットを利用すると直接Animationオブジェクトを渡して目的を達成できる。
    * PositionedTransition
    * RotationTransition
    * ScaleTransition
    * SizeTransition
    * SlideTransition
    * AlignTransition
    * DecoratedBoxTransition
    * FadeTransition
* 他
    * AnimatedModalBarrier
    * AnimatedList


