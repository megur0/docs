[TOP(About this memo))](../../README.md) >  [Flutter](../README.md) > [一覧(Flame)](./README.md) > Forge2D



#  Forge2D(forge2d)
* https://github.com/flame-engine/forge2d
* https://pub.dev/packages/forge2d
* Box2D 
  * オープンソースの物理エンジン
  * C++で書かれている
* Forge2D 
  * Box2DがDartに移植されたが途中からメンテナンスされずPRが取り込まれなかったため、別名のForge2Dとして開発されるようになった
  * 現在もともとのDart移植版のリポジトリ(box2d)はアーカイブされている
    * https://github.com/google/box2d.dart

# flame_forge2d
* https://github.com/flame-engine/flame/tree/main/packages/flame_forge2d
* https://pub.dev/packages/flame_forge2d
* forged_2dとFlameゲームエンジンのブリッジを行うパッケージ
* forge2dに依存する
* 実装自体は小さなコードベースとなる。

# サンプルコード
* https://github.com/flame-engine/flame/tree/main/examples/lib/stories/bridge_libraries/flame_forge2d
* シンプルな完全なサンプル
  * ボールがバウンドするサンプル
  * https://github.com/flame-engine/flame/tree/main/packages/flame_forge2d/example

# Box2D(Forge2D)の速度制限と、そのflame_forge2d側の対策について
* 詳しくは理解できていないが、Box2D, Forge2Dには速度制限があり、その速度制限に達するとそれより速度は上がらない。
* (参考)forge2Dの設定箇所
  ```dart
  // packages/forge2d/lib/src/settings.dart

  /// The maximum linear velocity of a body. This limit is very large and is used
  /// to prevent numerical problems. You shouldn't need to adjust this.
  double maxTranslation = 2.0;
  double maxTranslationSquared = maxTranslation * maxTranslation;
  ```
* (参考)Box2Dの速度制限
  * https://www.iforce2d.net/b2dtut/gotchas#speedlimit
  * https://stackoverflow.com/questions/14774202/is-there-an-upper-limit-on-velocity-when-using-box2d
  * https://siv3d.jp/bbs/patio.cgi?read=181
## flame_forge2d側の対策
* flame_forge2dでは、すぐにForge2Dの速度制限に達してしまうことから、その対策としてズームレベルを10.0としていると記載されている。
  * https://docs.flame-engine.org/latest/bridge_packages/flame_forge2d/forge2d.html
    > Forge2DGame has a built-in CameraComponent and has a zoom level set to 10 by default, so your components will be a lot bigger than in a normal Flame game.  
    >  This is due to the speed limit in the Forge2D world, which you would hit very quickly if you are using it with zoom = 1.0.
  * なぜズームする必要があるのか？
    * それ以外に対策が無いから？
    * Fixture自体を大きくする -> 見た目の加速が鈍くなる(速度の上限も遅くなる？)
    * 筆者もカメラ（ワールド）を使わずにサンプルコードを書き換えてみたが、速度やバウンドが鈍くなってしまった。
      * https://gist.github.com/megur0/9952cccb957ea2dbc34fd75b6155dd5d
* flame_forge2dではカメラをズームすることで速度制限に対応しているため、実質的に、下記のようにカメラを利用した実装をする必要がある。
  * ポジションの基準となるコンポーネントはワールドに配置する
    * ズームを適用するため
    * 相対的な位置に配置する子コンポーネントは直接親に配置する必要はない。
      * 例えば下記のサンプルではシミュレーションした動作を実現するBodyComponentへ子コンポーネントとしてSpriteComponentを設定している。
      * https://github.com/flame-engine/flame/blob/main/examples/lib/stories/bridge_libraries/flame_forge2d/sprite_body_example.dart
    * (IMO)ドキュメントには下記のように記載されているが、WeaponがPlayerに対して相対位置であるならば、world.add(Weapon())ではなく、add(Weapon())が正しいと考えられる。
      * https://docs.flame-engine.org/latest/bridge_packages/flame_forge2d/forge2d.html#bodycomponent
      > So instead of add(Weapon()), world.add(Weapon()) should be used (as below), and the Player should also of course initially be added to the world.
  * 各コンポーネントの大きさや位置はズームを考慮した値にする
    * 例えば、MediaQuery.of(context).sizeが800x350のビューポートに、割合として50x50の四角を配置するには、(ズームが10倍であれば)sizeは5x5で指定する必要がある。
    * テキストのfontSizeもズームの割合だけ元のサイズから小さくする必要がある
  * 画面の境界はカメラの範囲に基づく
    * 境界はgame.camera.visibleWorldRectの値を利用すれば良いだろう。
    * 以下の公式サンプルコードで壁の生成に利用されている
      * https://github.com/flame-engine/flame/blob/main/examples/lib/stories/bridge_libraries/flame_forge2d/utils/boundaries.dart


# flame_forge2dを利用する際に実装者がすること、その事由
* GameはForge2DGameを利用する
  * このクラスがworldとしてForge2DWorldを扱うため。
* 物理シミュレーションが必要なコンポーネントはBodyComponentを利用する
  * このクラスがonLoadの際に、Forge2DWorldを利用してBodyを生成して、forge2d.World.bodiesへ追加、Fixture(衝突検知のためのシェイプの情報)を適用して物理シミュレーションの対象になるようにするため。
  * このクラスがレンダリングの際に、物理シミュレーションの結果(forge2d.Bodyのpositionやangle)とFixtureをキャンバスへ適用するため。
* BodyComponent.createBodyをオーバーライドしてBodyを返すか、bodyDef, fixtureDefs(オプション)を(createBodyが呼ばれる前に)設定する
  * オーバーライドする場合、Bodyの生成はworld.createBodyを利用する。(これによって物理シミュレーションの対象となる)
* ポジションの基準となるコンポーネントはワールドへ配置する
  * zoomはデフォルトに従って10.0としておいた方が良いだろう
  * 事由は前述
* 接触時に物理シミュレーション以外の動作を発生させたい場合は、ContactCallbacksを利用する。
  * BodyComponentで実装を行えば、自動的にシミュレーションされたpositionやangleなどが更新される。
  * BodyComponent同士の接触時のシミュレーションも都度反映される。
  * その他の処理を接触時に発生させたい場合、ContactCallbacksを利用する。

# Forge2DGame
* FlameGameの派生クラス
* WorldはForge2DWorld派生クラスを利用する
* camera
  * zoomがデフォルトで10.0となる。

# Forge2DWorld
* BodyComponentと通常のFlameコンポーネントの両方を扱う。
* これがforge2D(forge2d.World)のラッパーとなる。
  * Forge2DWorldの各メソッドは、ほぼforge2d.Worldの各メソッドのラッパーとなっている。
* gravity
  * デフォルトはVector2(0, 10.0)
  * physicsWorld.gravityに設定される。Forge2DGame経由で設定できる。
  * Forge2DGameでは、Flameと同じ座標系を維持するために、Y軸および重力がForge2Dと反転している。(とドキュメントに記載してある)
    * Vector2(0, 10.0)の場合はボディは下方向に引っ張られ、負の値に設定すると上方向に引っ張られる。
    * TODO: 実装を見たが、どこで反転しているのかは具体的に分からなかった。
      * Forge2DGame -> Forge2DWorld -> forge2d.World と渡す際に反転している箇所はなかった
      * BodyComponent.renderTree にもそれに関する記述は無かった
* update
  * forge2d.World.stepDtを実行してforge2d.World.bodiesの物理シミュレーションを実行する。
* world
  * forge2d.World
  * 物理的なエンティティやシミュレーション、非同期なクエリを管理するクラス
  * メモリ管理の機能も保有する
  * bodies
    * Bodyのリスト
    * 各BodyComponentがForge2DWorld.createBodyを呼んでBodyを生成するとともに、このbodiesに加えられてシミュレーションの対象になる。
  * fixtures
    * Fixtureのリスト
    * FixtureはBodyにShapeをアタッチするために利用され、これは衝突の検知に利用される。
    * 各BodyComponentからBody.createFixtureが呼ばれることで、bodyへ適用されるとともに、fixturesへ追加される。
  * contactListener
    * デフォルトでWorldContactListenerが設定される
    * WorldContactListenerはforge2d.ContactListenerの派生クラス
      * https://pub.dev/documentation/flame_forge2d/latest/world_contact_listener/WorldContactListener-class.html
      * 接触の際にmixinのContactCallbacksのメソッドを呼ぶためにこの派生クラスが設定されている。
      * 少しトリッキーで、衝突時のコールバックで渡されるContactから取得できるユーザーデータ（Object型）に格納されたContactCallbacksオブジェクトを取り出してメソッドを呼び出している。

# BodyComponent
* HasGameReference<T extends Forge2DGame>, HasPaint を 既に withしている
  * したがって、デフォルトでgameを参照できる。
  * paintに値を設定することで描画される色や線などを設定できる。
  * また、world, cameraプロパティがあり、 game.world, cameraを参照する
* 座標
  * BodyComponentはアンカー指定がなく、常に自身の真ん中が、論理的な中心となる。
* postion
  * body.positionを指す
  * bodyの生成時にBodyDefの引数として渡すことで初期値を設定する。body.positionのデフォルト値はVector2.zero()
* 子コンポーネント
  * 他のコンポーネントと同様に子コンポーネント(SpriteComponentなど)を持たせることができる。
* 以下のどちらかの実装を行う必要がある。
  * createBodyをオーバーライドしてforge2d.Bodyを返す
    ```dart
    // 下記のようにworld.createBodyを呼び出してBodyを生成する
    @override
    Body createBody() {
      final fixtureDef = FixtureDef(
       //...
      );
      final bodyDef = BodyDef(
        //...
      );
      return world.createBody(bodyDef)..createFixture(fixtureDef);
    }
    ```
  * bodyDef, fixtureDefsを(createBodyが呼ばれる前に)設定する。(コンストラクタ等)
    * bodyDef
      * bodyを作成するためのパラメータ
    * fixtureDefs
      * bodyに適用されるフィクスチャ
      * friction: 摩擦係数 0〜1の範囲
      * restitution: 反発係数 0〜1の範囲
      * shape: PolygonShape, CircleShape, EdgeShape, ChainShape
    ```dart
    final shape1 = EdgeShape()..set(Vector2(0, 0), Vector2(10, 10));
    final shape2 = PolygonShape()..set([
      Vector2(-10, 5),
      Vector2(10, 5),
      Vector2(0, -5),
    ]);
    final shape3 = PolygonShape()..setAsBoxXY(someWidth * 0.5, someHeight * 0.5);
    final fixtureDef = FixtureDef(
      shape1,
      userData: this, // To be able to determine object in collision
      restitution: 1.0,
      friction: 0.0,
    );
    //...
    ```
* createBody
  * onLoadにて呼び出され、body(forge2d.Body)にセットされる
  * デフォルトの実装
    ```dart
     Body createBody() {
      assert(
        bodyDef != null,
        'Ensure this.bodyDef is not null or override createBody',
      );
      final body = world.createBody(bodyDef!);
      fixtureDefs?.forEach(body.createFixture);
      return body;
    }
    ```
  * Forge2DWorld.createBodyによって、forge2d.World.bodiesに追加されることで、物理シミュレーションの対象となる。
* type
  * BodyType.static, BodyType.kinematic, BodyType.dynamic
  * 少し理解が怪しいが、おおざっぱに下記のように理解している
  * staticの場合は通常は移動しない。衝突した際は速度のパラメータに応じて移動する
  * kinematicは移動はあるが、衝突による影響を受けない
  * dynamicは速度に基づいて移動し、衝突すればその影響を受ける
* world 
  * => game.world
* renderTree
  * body.positionやbody.angleをcanvasへ適用している
* render
  * 各 body.fixtures 毎に処理が行われる
  * Fixture.type ごとにメソッドが用意されており、オーバーライド可能
    * ShapeType: chain, circle, edge, polygon
    * renderXXX
  * body.fixturesに基づいてではなく、独自に描画したい場合は renderメソッドをオーバーライドすれば良い。
* renderBody
  * falseにすると描画が行われない。
  * 注意: debugModeがtrueとなっている場合は、renderBodyがfalseでもrenderDebugModeから描画が行われる。
    ```dart
    @override
    void renderDebugMode(Canvas canvas) {
      body.fixtures.forEach(
        (fixture) => renderFixture(canvas, fixture),
      );
    }
    ```
* onRemove
  * ボディのdestroyが呼ばれている。
* renderer
  * 描画をするか否か。デフォルトはtrue
  * 例えば、子のコンポーネントとしてスプライトやテキストコンポーネントを設定する場合などは、自身は描画する必要がないためfalseとするといった用途がある。

# forge2d.Body
* https://pub.dev/documentation/forge2d/latest/forge2d/Body-class.html
  > A rigid body. These are created via World.createBody.
* postion
  * デフォルト値はVector2.zero()となる
* setActive
  * フラグ(Body.flags)のアクティブ状態を変更する
  * 非アクティブの場合、シミュレーションが止まりその場で停止する。(他のBodyとの衝突が発生せず、速度による移動も発生しない)
  * 非アクティブ -> アクティブ状態となると、動作が再開する
* setAwake
  * フラグ(Body.flags)のawake状態を変更する
  * 筆者の解釈では、
    * setAwake(true)とすると、速度や力(force)などがリセットされ非awake状態となる
    * 非awake状態の場合は速度によるシミュレーションが行われず、衝突は発生する。
* linearVelocity
  * 位置の移動速度
* force
  * 力
  * 衝突などが発生した際に力が発生すると考えられる
* angularVelocity
  * 回転速度
* applyLinearImpulse
  * 衝突のエネルギーを発生させて位置の移動速度を変化させる。
* applyForce
  * 力を発生させる
* isBullet
  * 弾丸のようなボディとして扱われるかどうか
  * TODO
* setMassData
  * 質量の設定
  * TODO

# mixin ContactCallbacks
* BodyComponentが接触した際のイベントを実装するために利用する
* 注意点として、forge2d.Body.userDataにContactCallbacksオブジェクトを設定する必要がある。
  * https://docs.flame-engine.org/latest/bridge_packages/flame_forge2d/forge2d.html#contact-callbacks
  * 設定する際は、FixtureDefもしくはBodyDefのuserDataに設定してBodyを作成(Bodyへ適用)すれば良い。
  * どちらか片方に設定すれば良い。
  * 設定したオブジェクトは、衝突の際に検知するオブジェクトとなる。
  ```dart
  final fixtureDef = FixtureDef(
    userData: this,
    //...
  );
  ```
  ```dart
  final bodyDef = BodyDef(
    userData: this,
    //...
  );
  ```
  ```dart
  @override
  void beginContact(Object other, Contact contact) {
    if (other is XXX) {// 上記で設定したthisが、ここでコールバックで受け取るオブジェクトとなる。
      //...
    }
  }
  ```

# 通常のFlutterウィジェットを物理シミュレーションで動かす
* 以下の方法で実現する
  * GameWidget.overlayBuilderMapを利用して通常ウィジェットをGameへ被せる
  * BodyをGame内で生成し、overlayBuilderMapからの引数gameを経由してウィジェットからBodyを参照する
  * ウィジェットはBodyに基づいてPositionedやTransformで配置する
  * gameにコールバックを渡してTick毎にsetStateさせる
* https://github.com/flame-engine/flame/blob/main/examples/lib/stories/bridge_libraries/flame_forge2d/widget_example.dart
* 参考
  * https://stackoverflow.com/questions/69328474/can-i-use-a-widget-as-a-actual-component-in-flutter-flame
   

# (未読了)Joints
* https://docs.flame-engine.org/latest/bridge_packages/flame_forge2d/joints.html
> Joints are used to connect two different bodies together in various ways.   
> They help to simulate interactions between objects to create hinges, wheels, ropes, chains etc.
* BodyComponentの位置(body.position)は手動で設定できないが、MouseJointDefを使ってドラッグ操作についてくる実装は可能
  * https://github.com/flame-engine/flame/blob/main/examples/lib/stories/bridge_libraries/flame_forge2d/utils/boxes.dart#L50
  * 参考
    * https://stackoverflow.com/questions/19924396/drag-box2d-bodies-on-touch
