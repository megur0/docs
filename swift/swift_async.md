---
title: "非同期処理 - Swift"
updated: 2026-07-26
---

[TOP(About this memo))](../README.md) > [一覧(Swift)](./README.md) > 非同期処理


## PromiseKit、Combine、Swift Concurrencyについて
* (参考) https://tech-blog.optim.co.jp/entry/2022/03/25/100000
* コールバックによる非同期処理
    * 可読性が悪い。
* PromiseKit
    * いわゆるPromise(Future)パターンをSwiftで利用可能にするOSSライブラリ。
    * thenやdoneなどで処理をチェインさせる構造。
    * Swift Concurrencyが登場したことで、あまり使われなくなっていく可能性がある(?)。
* Combine
    * iOS13以降で使用可能となった、Reactive Programmingな実装を実現するためのフレームワーク。
    * 多くのCombineを使った非同期処理は、Swift Concurrencyで置き換え可能。
    * 状態管理のObservableObjectプロトコルや@PublishedなどのProperty Wrapperとして間接的にCombineが使われているだけで、それ以外は基本的にSwift Concurrencyを使う形になっていくと考えられる(?)。
* Swift Concurrency
    * Swift 5.5で追加された非同期・並行処理関連の機能群。


## RxSwiftやReactiveSwift
* サードパーティ製ライブラリ。
* 非同期処理やイベント処理のためのライブラリ。
* Combine、Swift Concurrencyの登場によって、今後の使用が減少していく可能性がある(?)。


## Combine
* (参考) https://www.bravesoft.co.jp/blog/archives/15610
* iOS13から追加されたApple純正のフレームワーク。
* 簡単に言うと、イベントを発行する側と受け取る側に分かれて、あるイベントが発行されたら、それを受け取った側の処理が走ることを容易にしたフレームワーク。
* ObservableObjectプロトコルや@PublishedなどのProperty Wrapperは、実はCombineの機能。
    * Foundation経由で使われているため、Combineを直接importしなくても使えている。
* Publisher -> Operator（再発行者）-> Subscriber
* Subject
```swift
import Combine
var cancellable: AnyCancellable?
let subject = PassthroughSubject<Int, Never>()
cancellable = subject.sink { num in print(num) }
subject.send(1)
subject.send(2)
cancellable?.cancel() // キャンセルすると
subject.send(3)
```
* sink以外にも、assignを使うとPublisherから流れてくるデータをオブジェクトにバインディング（代入）できる。
* Future
    * Futureは非同期で一つの値を生成して出力するか失敗するPublisher。
    * これまでの書き方
    ```swift
    import Foundation
    print("2秒後に「HOGE」が出力されます")
    performAsyncAction {
        print("HOGE")
    }
    func performAsyncAction(completionHandler: @escaping () -> Void) {
        DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
            completionHandler()
        }
    }
    ```
    * Futureを使った書き方
    ```swift
    import Foundation
    import Combine
    var cancellable: AnyCancellable?
    print("2秒後に「HOGE」が出力されます")
    cancellable = performAsyncActionAsFuture().sink { _ in
        print("HOGE")
    }
    func performAsyncActionAsFuture() -> Future<Void, Never> {
        return Future() { promise in
            DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
                promise(Result.success(()))
            }
        }
    }
    ```
    * （参考）async/await
    ```swift
    import Foundation
    import Combine
    import _Concurrency // Playgroundで実行させる時に必要。
    print("2秒後に「HOGE」が出力されます")
    Task {
        let result = await performAsyncActionAsAsyncAwait()
        print(result)
    }
    func performAsyncActionAsAsyncAwait() async -> String {
        try? await Task.sleep(nanoseconds: 2_000_000_000)
        return "HOGE"
    }
    ```


## Swift Concurrency
* コールバックを使う方法は可読性が悪い。
* Swift 5.5からは、Swiftの言語機能としてSwift Concurrencyが登場。
    * 非同期処理、並行処理のコードを簡潔かつ安全に記述できる機能。
    * async/await、Task。
* asyncなメソッドを呼べる箇所
    * 非同期な関数やメソッドの中。
    * mainメソッドや@mainがつけられた場所。
    * detached Taskやunstructured Taskの中。
* Task
    * awaitを使用する際、asyncなメソッド外から呼び出す場合はTaskで括る必要がある。
* unstructured Task（`Task{}`）と detached Task（`Task.detached{}`）
    * Appleとしてはunstructured Taskのほうが推奨(?)。
    * unstructured Taskは実行コンテキストを引き継ぐが、detached Taskは実行コンテキストを引き継がない。
    * わかりやすく言うと、この2つのTaskは開始時の実行スレッドが異なる。unstructured Taskは、そのTaskが呼ばれた場所の実行スレッドをそのまま引き継ぎ、detached Taskは引き継がない。
    * unstructured Taskは、メインスレッドから呼ばれればメインスレッドで実行され、バックグラウンドスレッドで呼ばれればバックグラウンドスレッドで実行される。
    * detached Taskはどのスレッドから呼ばれても、呼ばれたスレッド以外のスレッドで実行される。
    * `Thread.isMainThread`を使うと、その行がメインスレッドで実行されているかどうかがわかる。
* メインスレッドで行うべき処理
    * 基本的に、画面描画に関わる処理はメインスレッドで行わなければならない。画面に表示する文字列を変更したり、新しい画面を開いたり、画面に表示された画像を変更したり…あらゆる処理はメインスレッドで行う。
    * 時間のかかる処理をメインスレッドで行ってしまうと、画面の動きがもたついたり、ユーザーに対するレスポンスが遅くなってしまうため、バックグラウンドのスレッドで行う。
* MainActor
    * iOS13から登場したActorを使うと、どこから呼ばれてもメインスレッドで実行するように指定することができる。
    * クラス単位で@MainActorをつけておくと、そのクラスのすべてのメソッドがメインスレッドで呼ばれる（asyncなメソッドは除く）。
    * クラスの一部の処理だけメインスレッドで実行したい場合は、メソッドに@MainActorをつける。
    * なお、UIViewControllerには@MainActorがついているため、通常のメソッド（async以外）は常にメインスレッドで呼ばれる。
* (参考) https://www.toyship.org/2021/12/03/105551?amp=1
