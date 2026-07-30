---
title: "導入 - Swift"
updated: 2026-07-26
---

[TOP(About this memo))](../README.md) > [一覧(Swift)](./README.md) > 導入


## Swiftの特徴（勉強中）
* 値型中心の言語である。
    * (参考) https://heart-of-swift.github.io/value-semantics/how-to-use-value-types.html
    * 構造体にプロトコルを準拠させてインスタンス化するという書き方が特徴的。
* 機能が充実・洗練されている印象がある。
    * enum
    * エラーハンドリング
* プロパティラッパー
* アトリビュートが多い。
* クロージャ式（無名関数）
    * (IMO) 書き方に癖がある。
    * `{ (引数) -> 戻り値の型 in 処理 }` という書き方。`(引数)(戻り値の型){処理}` のような書き方でも良いのではと感じた。
    * さらにトレーリングクロージャという省略記法があり、慣れるまでは読み解くのが難しい（慣れれば便利かもしれない）。
* (IMO) 関数の戻り値の型宣言に `->` を使うため、戻り値や引数が関数型の場合に `->` が連続し、やや読みづらく感じる。


## わからないこと(?)
### ルートレベルで計算プロパティが書けるのはなぜか
```swift
protocol Animal {
    func sound()
}
struct Dog: Animal {
    func sound() {
        print("bow wow")
    }
}
var pet1 = Dog()
// var pet1: Animal = Dog() // これも上記と同義
// var pet1: Animal { Dog() } // なぜかこれも動作した（オンラインの実行環境）。これは変数ではなく計算プロパティという理解で、クラスや構造体の中でしか書けないと思っていたのだが…
pet1.sound() // bow wow
```

### Viewへの準拠時のルールでわからないこと
```swift
struct ContentView: View {
    /* var body: some View {
        Text("Hello, World!")
    } */
    var body: Text = Text("Hello, World!") // Text構造体はViewに準拠していないように（コードを読むと）見えるが、これでコンパイルが通る理由がわからない。
}
```


## (IMO) SwiftUI と Flutter の比較
* 内部構造がFlutterに比べて少しわかりづらいと感じた。
* 例えばFlutterのColumnは子ウィジェットを `List<Widget>` で保持しており構造が把握しやすいが、SwiftのVStackは内部でアトリビュートによる変換を行っていたり、List構造ではなくTupleViewのような構造を使っているため、ジェネリクスを多用した複雑な構造になっている。
* ただし、someやアトリビュートの変換処理によって、UIの記述自体はFlutterより簡潔になっている印象がある。


## チュートリアル
* https://developer.apple.com/tutorials/swiftui/creating-and-combining-views


## Swift Playground
* https://apps.apple.com/jp/app/id1496833156
* 手軽に実行するならこれで十分そう（非公式ではあるものの）。
    * (参考) https://paiza.io/projects/eOPjaLd3Odv8pZZdEFJirA?language=swift


## 公式ドキュメント
* SwiftUI
    * https://developer.apple.com/documentation/swiftui
* Swift
    * https://docs.swift.org/swift-book/documentation/the-swift-programming-language/


## swift.codelly
* わかりやすく体系的にまとまっている。
* (参考) https://swift.codelly.dev/


## Xcode
* フォーマット: `cmd + a` で全選択して `ctrl + i`
* ブレークポイントを設置できる。
* コードエリア右側の左下にある再生ボタン、または `command + option + p` で実行するとプレビューが表示される。
* 左上の再生ボタン、または `command + r` で実行するとシミュレーターが起動する。
