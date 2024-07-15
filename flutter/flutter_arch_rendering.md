[TOP(About this memo))](../README.md) > [一覧](./README.md) >




# Flutterのレンダリングモデル
* 一般的なクロスプラットフォーム フレームワークの場合
    * AndroidやiOS等のネイティブのライブラリ上に抽象化レイヤーを作成し、アプリコードと対話することでUIを表示
    * したがって表示するUIはネイティブのライブラリに則したものとなる。
* Flutterの場合
    * Dartコードは機械語にコンパイルされ、VM上で実行して直接デバイスのグラフィックエンジンを使って描画をする。
    * したがってウィジェットが独自のものとなっている。
    
# レンダリングエンジン
* Skia
    * Flutterは、モバイルのUIレンダリングエンジンとしてSkiaを採用している。
    * アプリ初回起動時にシェーダーコンパイルが行われるため、初回起動時のパフォーマンスが良くない。(60FPSを下回るなど)
        * Andoridはコンパイルの結果が永続化されるため、パフォーマンスが悪いのはアプリインストール後の起動時のみとなる。
        * iOSはタスクキルすると廃棄されるため、アプリを起動するたびにコンパイルが必要となる。
    * Skiaにはこのシェーダーコンパイルを事前にしておくSkSL(Skia Shader Language) warmupという機能があるが、開発者の負担が大きいという問題がある。
* Impeller
    * https://docs.flutter.dev/perf/impeller
    * Flutterはバージョン3.0.0でImpellerというUIレンダリングエンジン導入した。
    * iOSではデフォルトで有効になっているが、Androidではプレビュー版であり3.19リリース時点ではデフォルトでは無効となっている。

# 描画（レンダリング）の流れ
* 参考
    * https://zenn.dev/seya/articles/f7ebcd8335eee7
* Flutter Framework と Engine は繰り返しやり取りをしながら Flutter の描画が行われている。
    * フレームの間隔は Engine によって制御されている。
    * フレーム毎に Engine から Framework へ DrawFrame という関数を実行するように命令される。
    * そこから Framework の層ではWidget や RenderObject の更新をし、新しいレイアウトを再計算して最終的にまた Engine に描画のリクエストを行う。
* このやり取りは Binding と呼ばれるインターフェイスを介して行われる。

# (参考)(IME)フレーム処理の例
## 具体例：スクロール処理
|処理|内容|サイクル|
|-|-|-|
|スクロールイベントの監視|画面からのスクロールイベントの検知|イベントループ(実行環境のCPU性能依存?)|
|ディスプレイのリフレッシュ|表示するディスプレイのリフレッシュ。OSからシグナルがエンジンへ送られる。|ディスプレイのリフレッシュレートに依存|
|スクロール画面のビルド・リビルド|リビルド対象のウィジェットについてレンダリングツリーの再構築が行われる<br/>ウィジェット次第だが多くの場合リビルドされると直後に再レイアウト・レンダリングも行われる。|毎フレーム *1|
|スクロール画面のレイアウト・再レイアウト|リビルド対象のウィジェットのレイアウト（各ウィジェットのサイズと位置）の再計算が行われる|毎フレーム *1|
|スクロール画面のレンダリング・レンダリング|レンダリング対象のウィジェットの再描画が行われる|毎フレーム *1|

* *1 対象となるウィジェットが発生した際は、エンジンに対して次回のフレームにおいてレンダリングパイプラインを実行するdrawFrame()という関数の処理がスケジューリングされる。次回のフレームにてビルド・レイアウト・レンダリングの処理が実行される。
    * 実際の内部処理はもう少し細かいフェーズに分かれている。
    * https://api.flutter.dev/flutter/widgets/WidgetsBinding/drawFrame.html
* たとえば、下記のようにスクロールイベントごとにprint()を実行しつつ、末端に移動したらローディング表示をする実装をしたとする。
    ```
    final controller = ScrollController();
    controller.addListener(() {
        print("${controller.position.pixels}, ${controller.position.maxScrollExtent}");
        if (controller.position.pixels >= controller.position.maxScrollExtent) {
            // ロード処理開始（非同期のデータのフェッチなど）
            // ウィジェットがローディング表示をするためのフラグなどを変更する。
            // setState（リビルド対象となる）
        }
    });
    ```
    * スクロールイベントの監視
        * print()はイベントが検知される度に出力される。
        * この処理はフレーム処理とは異なり、一般的にサイクルは実行環境の性能に依存する。
    * スクロール操作による画面の再レイアウト・レンダリング    
        * スクロールによって発生した速度から次回フレームの座標の変化がシミュレーションによって算出され、再レイアウト・レンダリングの対象となる。
        * 次のフレームでエンジンによりレンダリング処理が行われる。
            * 適宜再レイアウトされる
            * レンダリングされる
            * これによってスクロール内のコンテンツやスクロールバーなどの移動が見た目上反映される。
    * setState()によるスクロール画面のビルド・リビルドおよび再レイアウト・レンダリング
        * setState()されることでリビルドの対象となり次のフレームでリビルドされる。その後再レイアウト・レンダリングされて画面にローディング表示が反映される。


# Bindingの種類
|種類|役割|
|-|-|
|SchedulerBinding|Engine と Framework　のやり取りをしている Frame の間隔の管理など|
|GestureBinding|ジェスチャー(画面をタップしたりとか)などのイベントを伝える役割|
|RendererBinding|Engine と Render Tree を繋ぐ役割|
|WidgetsBinding|Engine と Widget を繋ぐ役割|
|ServicesBinding|プラットフォームチャネルによって送信されたメッセージの処理|
|PaintingBinding|画像キャッシュの処理|
|SemanticsBinding|セマンティクスに関連するすべての後の実装のために予約|
|TestWidgetsFlutterBinding|ウィジェット テスト ライブラリによって使用される|
## (参考)WidgetsFlutterBinding.ensureInitialized() 
* runApp()を呼び出す前にFlutter Engineの機能を利用したい場合に使われる初期化処理。
* "全ての Binding を初期化する" という役割となる。
    * (参考)内部での処理ではmixin を利用して各Binding の　initInstancesメソッドを順に実行している。
* 公式ドキュメントには、「フレームワークとFlutterエンジンを結びつける接着剤のような役割」と述べられている。
    > This is the glue that binds the framework to the Flutter engine.
    * https://api.flutter.dev/flutter/widgets/WidgetsFlutterBinding-class.html


# Widget, Element, RenderObjectの３つの構造
* https://docs.flutter.dev/resources/architectural-overview#build-from-widget-to-element
* 3層構造はパフォーマンスのため。
    * 毎回対象のツリーを再計算するのは非効率となるため。
* Element と RenderObject にさらに分かれている理由 は、 パフォーマンス、明快さ、型の安全性、等とドキュメントでは述べられている。
* これによって下記を両立している。
    * ウィジェットツリーに変更を加わった際に完全に使い捨てであるかのように動作できる
    * 再構築ではなくキャッシュを利用
* WidgetはFlutter開発者がUIを構成するために利用するインターフェース。
* Elementは内部のツリー構造を実行時に構築する中心的な役割となる。
* RenderObjectは画面へ描画するための基となる情報となる。
* 最終的な画面への描画を担うのはRenderObjectのみであり、それ以外は描画に必要な情報をツリー上で伝搬・集約するための役割となる。


# Widget
* Flutter appはそれ自体がwidgetである。
* ウィジェットはイミュータブルである
    * 自身のUIの構成情報と子Widgetをfinalなフィールドとして保持する
    * StatefulWidgetもウィジェットクラス自体はイミュータブルである点に注意。
* 5種類のウィジェットがある
## RenderObjectWidget    
* 描画を行うウィジェットは、このRenderObjectWidgetを継承した何かしらのウィジェットを生成している。
    * (例)Textウィジェット 
        * StatelessWidgetを継承
        * RichTextウィジェットを内部で生成。
        * RichTextウィジェットは MultiChildRenderObjectWidget という RenderObjectWidget を継承したクラス
* RenderObjectElement 及び RenderObject をどのように生成するかの情報を含んでおり、見た目の部分に関わる。
* createElement()メソッド
    * Widget を継承したクラスがインスタンス化される時に実行される。
    * Element（RenderObjectElement）が作成される。
* createRenderObjectメソッド、updateRenderObjectメソッド
    * どんな RenderObject を作るのか/どうやってアップデートするのかを指定する
    * (参考)各ウィジェットがどのRenderObjectを生成するかは様々であり、実装を見てみると例えば以下のようなものがあった
        |ウィジェットクラスの例|createRenderObjectメソッドで生成されるRenderObject|
        |-|-|
        |RichText|RenderParagraph（extends RenderBox）|
        |TextButtonやIcon|RenderSemanticsAnnotations（extends RenderProxyBox）<br/>※ 正確にはbuildメソッド内で生成されるSemanticsウィジェットが生成する|
        |SizedBoxやConstrainedBox|RenderConstrainedBox（extends RenderProxyBox））|
* なお、RenderObjectWidgetを継承せずにCustomPainter という Widget をextends（implements）して描画を行っているウィジェットもある。
    * paint(Canvas canvas, Size size) メソッドによってCanvasクラスを使用して描画する。
    * このCustomPainterを使うやり方は、ユーザー自身で独自のウィジェットを使いたい時にも利用できる。
## StatelessWidget
* StatelessWidgetは状態の変更を伴わないウィジェットとなる。
* これらは内部で変数を更新したりそれをUIに反映することはできない。
* 例えばCard、DataTable、Container、Text、GestureDetectorなどがこのウィジェットを継承している。
## StatefulWidget
* StatefulWidgetオブジェクト自体は 変更不可（immutable） であり、ステートレスである。
* createState という State を返す関数を必ず override させる。
* このStateクラスに開発者は状態をもたせることができる。
* Stateクラスは以下のように次フレームにてUIを再ビルドさせる機構を持っている。
    * StatefulElementクラスへの参照を持っている。
    * setState関数でリビルドのフラグをダーティにする。
        * 内部ではStatefulElement.markNeedsBuildメソッドを呼び出している。
    * dirtyな Element は フレーム毎にてまとめて再ビルドされる。
* 例えば、Scaffold、TextField 等の 状態を保持するが StatefulWidget を継承している。
## ProxyWidget
* データを伝播するためのもの
* InheritedWidget、ParentDataWidget など
## _NullWidget
* フレームワーク内部で使われている。
* アプリケーションコードでは使わない。（使えない）


# Element
* Mutableなクラス。
    * 状態(State)を保持する
* widgets/framework.dartで定義されている。
* 以下の2種類がある
## ComponentElement
* RenderObjectは生成せず、Elementを作成・集約する機能をもつ
* 以下はComponentElementのサブクラスとなる。
    * StatelessElement
    * StatefulElement
    * ProxyElement、InheritedElement
## RenderObjectElement
* RenderObjectを生成し、その参照を持つ
* mount()メソッド
    * WidgetクラスのcreateElementメソッドの直後に呼ばれる。
    * attachRenderObject()メソッドを呼ぶ
* attachRenderObject()メソッド
    * Widgetから受け取ったRenderObjectをRenderObjectツリーに挿入
    * これによって、 Widget <-> Element <-> RenderObject という3層の構造が生成される。
## ElementはBuildContextによって抽象化されている
* Flutterのウィジェットはbuildメソッド等でコンテキスト情報を参照することで、例えば先祖のデータや設定などを参照できる仕組みになっている。
* BuildContextクラスの実体 は Elementとなる。
* ElementはBuildContext（抽象クラス）を実装している
* Elementの実装を隠蔽させるために BuildContextというインターフェースを間に入れている。
    * 開発者はBuildContextが提供するメソッドしか触れないため誤ってElementツリーを書き換えてしまうような危険な操作はできない。
    * なお、下記のように型アサーションで Elementとして直接使うことも可能。
        * (context as Element).visitChildren()
* 参考
    * https://github.com/flutter/flutter/blob/070c4943a732a66c87b3f22ae1569ba66c16d574/packages/flutter/lib/src/widgets/framework.dart#L3195
    * https://github.com/flutter/flutter/blob/070c4943a732a66c87b3f22ae1569ba66c16d574/packages/flutter/lib/src/widgets/framework.dart#L2123

# RenderObject
* Mutableなクラス
* UIの描画を行う
* RenderObjectツリーの情報をもとに、スクリーンにUIが描画される
* rendering/object.dartで定義されている
    * https://github.com/flutter/flutter/blob/21797cbb034f48a384378efff0ee0b520e160072/packages/flutter/lib/src/rendering/object.dart#L1237
* layoutメソッド
    * 引数に Constraintsを撮り RenderObject を描画する layout を計算する
* paintメソッド
    * 抽象メソッドとして定義され、各ウィジェットクラスでオーバーライドして描画処理を記述する。
    
# (参考)レンダーツリーの計算
* https://docs.flutter.dev/resources/architectural-overview#layout-and-rendering
* ボックス制約モデルは、O(n) 時間でオブジェクトをレイアウトする
* レイアウトを実行するために、Flutter は深さ優先のトラバーサルでレンダー ツリーをたどり、 親から子にサイズ制約を渡す。
* サイズを決定する際、子は親によって与えられた制約を尊重する。
* 子は、親が設定した制約内でサイズを親オブジェクトに渡すことで応答。
* 最後には、すべてのオブジェクトは親の制約内で定義されたサイズを持ち、メソッドを呼び出してペイントできる状態になる。

# (参考)WidgetsBinding.instance.addPostFrameCallback
* 実行にあたってビルドが完了している必要のある処理を行うために利用する。
* 例としてScrollControllerの操作がある。
    * ScrollControllerは、ListViewウィジェット等へ引数としてオブジェクトを渡すことで、コントローラー経由で様々な操作ができる。
    * ただ、InitState等の処理時点では、ウィジェットの初回ビルドは未完了でありスクロールコントローラーがスクロールウィジェットへ紐づけされていない。
    * スクロールのポジション情報（ScrollPostion)などはScrollControllerクラスから取得できるが、アタッチ（ScrollControllerのメンバーへ設定）が未済の場合はエラーとなる。
    * このとき、addPostFrameCallbackへコールバックを渡して確実にビルドされた後にこれらの処理を実行させることでエラーにならずにスクロール情報を扱った処理が実行できる。



