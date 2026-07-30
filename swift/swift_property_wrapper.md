---
title: "Property Wrapper - Swift"
updated: 2026-07-26
---

[TOP(About this memo))](../README.md) > [一覧(Swift)](./README.md) > Property Wrapper


## Property Wrapper
* プロパティの振る舞いを変更・追加し、簡単に再利用するための仕組み。
* lazyのように「プロパティの振る舞い」をいちいち言語の機能として組み込むと大変。
* 開発者がlazyと同じような「プロパティの振る舞いを変更」する機能を実装できるよう、Property Wrapperが導入された。
* 定義されたプロパティラッパーの例
    * @UserDefault
    * @Validated
    * @Binding
    * @ObservedObject
* 自分でプロパティラッパーを実装する場合
    * @propertyWrapperアノテーションを付与したstruct、class、enumとして宣言する。
* (参考) https://kobatech-blog.com/swift-property-wrapper/


## lazy
* プロパティを非Optionalとして公開しつつ、遅延初期化を実現できる。
```swift
struct Foo {
    lazy var foo = 1738
}
```
* lazyが無いと、オプショナルなプロパティをprivateで宣言してsetterを定義し、かつgetter内で初期値を返す・またはセットされた値を返すというコードが必要になり、ボイラープレートが増える。
* (参考) https://kobatech-blog.com/swift-property-wrapper/
