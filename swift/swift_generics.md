---
title: "ジェネリクス - Swift"
updated: 2026-07-26
---

[TOP(About this memo))](../README.md) > [一覧(Swift)](./README.md) > ジェネリクス


## 関連型（Associated Type）
* https://docs.swift.org/swift-book/documentation/the-swift-programming-language/generics/#Associated-Types-in-Action
* プロトコル定義時点では決められず、準拠側で指定したい型があるときに使う。
    * プロトコル用のジェネリクス的なもの（プロトコル自体にジェネリクス記法が使えない理由は不明(?)）。
* associatedtypeで指定する。
```swift
protocol Container {
    associatedtype ItemType
    mutating func append(item: ItemType)
    var count: Int { get }
    subscript(i: Int) -> ItemType { get }
}
struct IntStack: Container {
    var items = [Int]()
    typealias ItemType = Int  // 準拠させる側はtypealiasで指定。
    // 〜
}
```
* appendメソッドは引数にItemTypeを利用しなければならない。
* countはInt型を返さなければならない。
* subscriptによって返される型はItemTypeでなければならない。
* あまり書く機会は無さそう。プロトコルを書く必然が無い限り、連想型を使うこともないと思う。
    * そもそもプロトコルを書く機会自体が少ないだろう（読む機会は多いと思うが）。
* (参考) https://qiita.com/akeome/items/78e650f27a4c53e1406a
    * この例ではAPI処理の共通化を「プロトコルを定義して、そのextensionで共通処理を書く。各API処理ごとにプロトコルに準拠した構造体(URLやエンティティを設定)を定義しておいて、それを使って共通処理を呼び出す」という形で実現している。
    * その際、エンティティの型は準拠側で決定するので、プロトコル側では連想型にしておく形。
    * (IMO) プロトコルを使わずに、ジェネリクス関数で書くほうが分かりやすいかも？


## ジェネリクス
* (参考) https://blog.recruit.co.jp/rmp/mobile/studying-swift-generitics/
* (参考) https://qiita.com/shoheiyokoyama/items/31eca0d4b27bc9608eb8
* Generic Functions、型パラメータ(Type Parameters)
```swift
func xxxx<T>(a: T, b: T) {
    // 〜
}
```
* Generic Types
```swift
struct Stack<Element> {
    var items: [Element] = []
    mutating func push(_ item: Element) {
        items.append(item)
    }
    mutating func pop() -> Element {
        return items.removeLast()
    }
}
```
* 型制約（Type Constraint Syntax）
    * `<T: SomeClass, U: SomeProtocol>`
        * someTはSomeClassのサブクラスである必要がある。
        * someUはSomeProtocolに適合している必要がある。
* Where Clauses
    * `〜〜 where XXXX: YYYY`
        * XXXXがYYYYに準拠（サブクラス）である必要がある。
* 関連型（Associated Types）の制約
    * 関連型に制約を持たせることができる。
    * `where S.DynamicType == T`
* map関数を読み解く（(?) 下記は古いSwiftのwhere構文の例。Swift 3以降は型パラメータの`<>`内ではなく、関数シグネチャの後ろに`where`節を書く現在の書き方に変わっている点に注意）
    * `public func map<S: Dynamical, T, U where S.DynamicType == T>(dynamical: S, f: T -> U) -> Dynamic<U>`
    * mapという名前のメソッドである（`public func map`の部分）。
    * S型のdynamicalとT型の引数をとってU型を返す関数fを引数にとる（`(dynamical: S, f: T -> U)`の部分）。
    * 内部でU型が使われるDynamic型を返す（`-> Dynamic<U>`の部分）。
    * S型はDynamicalプロトコルに適合している必要がある（`S: Dynamical`の部分）。
    * S型の関連型であるDynamicTypeの型はTと同じ型である必要がある（`S.DynamicType == T`の部分）。
    * whereは「U」ではなく、「S: Dynamical, T, U」に係っていることに注意。
