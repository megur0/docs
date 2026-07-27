---
title: "状態管理 - Swift"
updated: 2026-07-26
---

[TOP(About this memo))](../README.md) > [一覧(Swift)](./README.md) > 状態管理



# 状態管理
* https://developer.apple.com/documentation/swiftui/managing-model-data-in-your-app
* (参考) https://zenn.dev/yorifuji/articles/swiftui-managing-model-data
* (参考) https://3.1415.jp/yc91jqwt
* (参考) https://zenn.dev/ueshun/articles/2b26aaad40d6a3
* Single Source of Truth: SSOT（信頼できる唯一の情報源）
    * ある事実（データ）を一カ所に集約し、コピーしないことによってデータの整合性を保つ設計概念。
* 状態の変更をViewに反映したい場合に、データ管理のProperty Wrapperを利用する。
    * SwiftUIでのViewはstructであるため、保持するプロパティを変更することができない。
    * データ管理のProperty Wrapperを付与することで、メモリ管理がSwiftUIフレームワークに委譲され、変更が可能になる。
    * データの変更が監視され、変更時に宣言されたViewのbodyが再描画される。
* @State、@Binding
    * @Stateと@Bindingは値型のデータを扱うProperty Wrapper。
* @State
    * (参考) https://shuhey-hashimoto.com/swiftui/state-observedobject-environmentobject-%E3%81%AE%E4%BD%BF%E3%81%84%E5%88%86%E3%81%91-swiftui/
    ```swift
    struct ContentView: View {
        @State private var text: String = "Hello"

        var body: some View {
            VStack {
                Text(text)
            }
            .font(.largeTitle)
        }
    }
    ```
    * 変更は監視され、変更があればbodyが再描画される。
    * @Stateで宣言されたプロパティは、そのViewとプロパティを参照する配下のViewからしかアクセスできない。
    * Appleは@Stateと一緒にprivate修飾子をつけることを推奨している。
    * 外部から値の設定ができないため、初期値の設定が必要。
    * イニシャライザで初期値を設定する場合は以下のようにする必要がある（普通に代入しても無視される）。
        * プロパティ名の先頭に`_`をつける。
        * `State(initialValue: 値)`を使って設定する。
    ```swift
    init() {
        _text = State(initialValue: "Hello")
    }
    ```
    * 値を変更するアクセスでは`$`を付与する。
        * `TextField("", text: $subject)`
* @Binding
    * @Stateで指定したプロパティを子Viewでも利用したい場合は、そのまま変数を渡せばよい。
        * この場合、変更が親側で発生した際は、親のbodyごと子も再描画されると理解している。
        * なお、子側でも対象のプロパティに@Stateをつけた場合は、子側も情報源となり、親側とは別物の情報源になるため、親側の変更が子側に反映されない。
    * 子側でこのプロパティを変更したい場合は、親側から`$`をつけて渡し、子側で@Bindingとして定義する。
        * このとき、子側の更新によって親側も再描画される（親ごと再描画されるということ）。
    ```swift
    struct SampleView: View {
        @State var name: String
        var body: some View {
            VStack {
                Text(name)
                AnotherView(name: $name) // $を付けると Binding になる
            }
        }
    }
    struct AnotherView: View {
        @Binding var name: String // Binding として定義する
        var body: some View {
            TextField("name", text: $name)
        }
    }
    ```
* @StateObject、@ObservedObject、@EnvironmentObjectは参照型のデータを扱うProperty Wrapper。
    * これらのデータオブジェクトをSwiftUIが監視できるようにするには、データオブジェクトがObservableObjectプロトコルに準拠する必要がある。
* ObservableObject
    * 監視したいプロパティに@Publishedを指定する。
    * ObservableObjectおよび@PublishedはSwiftの機能。
    ```swift
    class DataSource: ObservableObject {
        @Published var counter = 0
    }
    ```
    * なお、@Published、ObservableObjectはSwiftUIではなくSwiftの機能（Combine）。
        * CombineとはiOS13から使えるApple純正のフレームワーク。
    * ObservableObjectの動作時の流れ
        * ObservableObjectに準拠させたクラスは、objectWillChangeというプロパティ(Publisher)を持つ。
        * @Publishedで指定されたプロパティに変化が起こる直前にobjectWillChangeに通知され、objectWillChangeがPublisherとして変化を通知する。
    * 通知条件を制御する方法
        * (参考) https://qiita.com/chocoyama/items/a588c3569b8dd89cd223
        * @Publishedを使わずに手動でsendメソッドを呼ぶ。例えば、プロパティのwillSetで、特定の条件を満たしたときのみsendする等。
* @ObservedObject
    * (参考) https://ios-docs.dev/swiftui-observerdobjects/
    * プロパティの変更を監視するようSwiftUIに通知する。
    * ObservableObjectに準拠したclassのオブジェクトに対して、@ObservedObjectを指定する。
    ```swift
    @ObservedObject private var dataSource = DataSource()
    ```
    * 指定されたプロパティは信頼できる情報源となる。
    * 子Viewと共有する場合は、インスタンスを渡す。子View側でも@ObservedObjectを指定する必要がある。
    * 一部分のプロパティだけ共有する場合は、`$`と@Bindingを使う。
    * ライフサイクル
        * 親（先祖）Viewのbodyが更新される際に、破棄＆再生成される。
        * Viewのライフサイクル（Viewの生成、破棄）には紐づかないため、Viewの表示時に再生成されないし、Viewの非表示時にも破棄されずに残るので注意。
    * 具体例
        * 親: @StateObject（自身で更新）、子: @ObservedObject（自身で更新）
            * 子が更新した状態は、親が@StateObjectのプロパティを更新した時点（親のbodyが再描画される時点）で破棄＆生成されてしまう。
            * そもそも、@ObservedObjectではなく@StateObjectを使うのが適切。
        * 親: @StateObject（自身で更新）、子: @ObservedObject（親から受けとり、自身でも更新）
            * 子が更新した状態は親にも伝搬し、親のbodyが更新される。したがって、この際に子の状態が再生成されても更新後の状態を親から受け取るため問題ない。
            * 親側で更新した場合も同様。
        * 親: @StateObject（自身で更新）、子: @ObservedObject（親から受けとる。自身では更新しない）
            * 上記と同様に特に問題なし。
        * (参考: iOS13以前の場合) 親の親（ラッパー）: @State、親: @ObservedObject（自身で更新）、子: @ObservedObject（親から受けとる）
            * iOS13以前の場合は@StateObjectが使えないため、この方法になる（ややトリッキーではある）。
            * @Stateのラッパーを入れることで、先祖のbodyが更新された際でも@Stateを持っているView側（ラッパー）で状態を維持できる。
            * なお、@Stateは参照型に使うと変更監視は使えないが、メモリ管理はSwiftUIで行われる。
* @StateObject
    * iOS14から導入されたProperty Wrapper。
    * @Stateと同様に、そのViewとプロパティを参照する配下のViewからしかアクセスできない。
    * @StateObjectでマークしたインスタンスは信頼できる情報源となる。
    * ライフサイクル
        * Viewが表示されてから(onAppearが呼ばれてから)非表示になる(onDisappearが呼ばれる)まで。
        * つまり、Viewが表示した際に生成され、非表示になると破棄される。
* 参考
    * @ObservedObjectと@StateObjectの違い
    * (参考) https://qiita.com/0102_0102johnny/items/fba13b2e7cfa61d437e9
    * (参考) https://qiita.com/junzai/items/c609e2ded02733887341
* @EnvironmentObject
    * 階層をまたいで子Viewに渡したいときに使う。
    * 先祖側で、environmentObjectメソッドで渡す。
    * オブジェクトを利用したい子Viewは、@EnvironmentObject属性のプロパティを宣言することでインスタンスにアクセスすることができる。
