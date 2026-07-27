---
title: "SwiftUI"
updated: 2026-07-26
---

[TOP(About this memo))](../README.md) > [一覧(Swift)](./README.md) > SwiftUI


# SwiftUI
* (参考) https://zenn.dev/rikutosato/books/6cee0a2b8aa796/viewer/876e85
* (参考) https://zenn.dev/coconala/articles/d721a5f0edca6e
* SwiftUIは宣言的UIで、コードベースでレイアウトを組む。Storyboardを使わない。
* iOS13以降でしか使えない。
* プレビュー機能が存在する。
* Storyboard開発
    * オブジェクトをドラッグ&ドロップで配置してレイアウトを作っていく開発方法。
    * 2019年にSwiftUI開発が登場したことにより、この開発方法は主流ではなくなった(?)。
* SwiftUIからUIKitの部品を使うためのラッパーとしてUIViewRepresentable、UIKitからSwiftUIを使うUIHostingControllerなども用意されている。


# Viewプロトコル
* (参考) https://tech.amefure.com/swift-protocol
* 読み取り専用のコンピューテッドプロパティであるbodyプロパティのみを持つ。


# よく使うもの
```swift
Image
    Image("turtlerock").clipShape(Circle()).overlay { Circle().stroke(.gray, lineWidth: 4) }.shadow(radius: 7)
Text
VStack
HStack
Spacer
```


# modifier
```swift
.frame(height: 300)
.padding(.bottom, -130)
.foregroundColor(.green)
.font(.subheadline)
```
* 大元となっているのはViewプロトコルに定義されているmodifierメソッド。
* SwiftUIではこのメソッドを拡張してさまざまなモディファイアが定義されている。
* modifier(モディファイア)を自作する場合
    * ViewModifierに準拠した構造体を作成し、以下のどちらかの方法で呼び出す。
        * modifierメソッドで呼び出す（書き方がやや冗長になる）。
            * `Text("Hello").modifier(定義したモディファイア())`
        * Viewを拡張してメソッドを増やす（こちらがおすすめ）。
* (参考) https://tech.amefure.com/swift-modifier-create


# ForEach
* range型を渡して繰り返しViewを表示させることができる。
```swift
struct ContentView: View {
    let animals = ["カピバラ", "アルパカ", "トナカイ", "アルマジロ"]
    var body: some View {
        Form {
            ForEach(0..<animals.count) { num in
                Text(self.animals[num])
            }
        }
    }
}
```


# SwiftUIのコード
* インターフェースを定義して、それをextensionでたくさん拡張している。例えば下記。
    * 要件を追加
        * 例えばextension Viewで、foregroundColorなら下記のように書かれている。
        ```swift
        @inlinable public func foregroundColor(_ color: Color?) -> some View
        ```
    * 型メソッドを実装。
* 直接プロトコルに書かずに、わざわざextensionで分けているのは、(IMO) @available(iOS 13.0, macOS 10.15, tvOS 13.0, watchOS 6.0, *)のように、機能ごとに動作バージョンを分けるためではないかと推測している。
## 描画処理
* Viewプロトコルにはユーザーが実装すべき内容が書いてあるが、それを描画するレイヤがどこにあるかは分からない。
* (IMO) ユーザーがViewプロトコルに準拠したstructを実装し、それをPreviewProviderに渡すことで、Xcodeのランタイムが解析してプレビューしてくれている、という理解でいる。


# SwiftUIの省略記法の解説
* (参考) https://kumamotone.hatenadiary.jp/entry/2019/08/09/075526
```swift
var body: some View {
    VStack {
        Text("Turtle Rock").font(.title)
        Text("Text").font(.subheadline)
    }
}
```
これは、下記と同義。
```swift
var body: VStack<TupleView<(Text, Text)>> = VStack<TupleView<(Text, Text)>>(
    content: { () -> TupleView<(Text, Text)> in
        return ViewBuilder.buildBlock(
            Text("Turtle Rock").font(.title),
            Text("Text").font(.subheadline)
        )
    }
)
```
## 解説
まず、SwiftUIのように次々とViewを継ぎ足していくたびに、`VStack<TupleView<(Text, Text)>>`のような型指定を変更するのは苦行になってくる。
それをsomeを使って省略する。
```swift
var body: some View {
    VStack(
        content: { () -> TupleView<(Text, Text)> in
            return ViewBuilder.buildBlock(
                Text("Turtle Rock").font(.title),
                Text("Text").font(.subheadline)
            )
        }
    )
}
```
まずは、省略記法（トレーリングクロージャ）を使う。
```swift
var body: some View {
    VStack {
        ViewBuilder.buildBlock(
            Text("Turtle Rock").font(.title),
            Text("Text").font(.subheadline)
        )
    }
}
```
そして、SwiftUIではresultBuilderというSwiftの機能が使われていて、contentで指定した内容を変換してくれるので、下記のように省略して指定することができる。
```swift
var body: some View {
    VStack {
        Text("Turtle Rock").font(.title)
        Text("Text").font(.subheadline)
    }
}
```


# Grid
* Grid&GridRowは、iOS16以降で使用可能。


# ナビゲーション

## NavigationView
* iOS16以降はDeprecated。

## NavigationStack
* iOS16以降のみ使える。
* NavigationViewの課題
    * NavigationLinkを使って簡単にスタックの遷移ができるものの、コードで遷移先を制御するにはコードが複雑化する。
* NavigationStackによって、NavigationViewの課題が解決される。
    * 遷移先情報を宣言的に記述できる。
    * 遷移情報を配列で管理できる。
* (参考) https://zenn.dev/chocoyama/articles/32d52cf66dd6ff

## TabViewとNavigationStackを組み合わせる
* (参考) https://notificare.com/blog/2022/11/25/a-better-tabview-in-swiftui/
