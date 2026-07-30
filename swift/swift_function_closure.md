---
title: "関数・クロージャ - Swift"
updated: 2026-07-26
---

[TOP(About this memo))](../README.md) > [一覧(Swift)](./README.md) > 関数・クロージャ


## クロージャ
* https://docs.swift.org/swift-book/documentation/the-swift-programming-language/closures/
* クロージャは次の3つの形式のいずれかである（つまり、普通の関数もクロージャに含まれる）。
    * グローバル関数（普通の関数のこと）は、名前を持ち、値をキャプチャしないクロージャ。
    * ネストされた関数は名前があり、それを囲んでいる関数から値を取得できるクロージャ。
    * クロージャ式は、周囲のコンテキストから値を取得できる軽量構文で記述された名前のないクロージャ。


## 関数
```swift
func someFunc(bbb: String) -> [String] {
    // 〜〜〜
    return contents
}
```
* 第一級関数（高階関数）として扱える。
* ネスト可能。
* 複数の戻り値が可能。
* ステートメントが一つの場合はreturnを省略可能。
* 引数ラベルの省略
```swift
func addTwoInts(a: Int, b: Int) -> Int {
    return a + b
}
print(addTwoInts(a: 3, b: 8))
```
は以下のようにラベル省略可にできる。
```swift
func addTwoInts(_ a: Int, _ b: Int) -> Int {
    return a + b
}
print(addTwoInts(3, 8))
```
* 型宣言
```swift
func addTwoInts(a: Int, b: Int) -> Int {
    return a + b
}
var mathFunction: (Int, Int) -> Int
mathFunction = addTwoInts
print(mathFunction(3, 8)) // (?) 型宣言した変数経由で呼ぶ場合はラベルが使えなくなる？
```
### 参照渡し
* inoutキーワードを引数に設定することで、いわゆる「参照渡し」を実装できる。
* ただし、基本的にはあまり使う機会は無いだろう。


## クロージャ式（無名関数）
* https://tea-leaves.jp/swift/content/%E9%96%A2%E6%95%B0
* 波かっこ`{}`で囲った中に処理を記述する。
* 中に記述した処理を1つの単位として扱うことができ、変数に格納したり、関数などの引数に渡すことも可能。
```swift
{ (引数) -> 戻り値の型 in
    処理
    return // ステートメントが1行の場合、returnを省略することができる。
}
```
* 省略形
    * 引数なし、戻り値void
        ```swift
        { () -> () in print("foo") }
        ```
    * さらに`() -> ()`も省略できる
        ```swift
        { print("foo") }
        ```
* 即時実行
```swift
{ print("foo") }()
```
* 型指定
```swift
let f: (Int, Int) -> String
f = { (num1: Int, num2: Int) -> String in return String(num1 + num2) }
```


## 高階関数へクロージャ式を渡す
* https://tea-leaves.jp/swift/content/%E9%96%A2%E6%95%B0
* https://docs.swift.org/swift-book/documentation/the-swift-programming-language/closures/
```swift
func calculate(_ a: Int, _ b: Int, _ function: (Int, Int) -> Int) -> Int {
    return function(a, b)
}
calculate(10, 20, {
    (val1: Int, val2: Int) in
    return val1 * val2
})
```
* 関数の中で実行される関数を「コールバック関数」と呼んだりもする。
* 引数の数と戻り値の型は関数の宣言から類推されるので、次のように型の指定を省くこともできる。
```swift
calculate(10, 20, {
    val1, val2 in
    return val1 * val2
})
```
* さらに簡略化
```swift
calculate(10, 20, { $0 * $1 })
```
* さらにこんな書き方もある。
```swift
calculate(10, 20, *)
calculate(10, 20, +)
calculate(10, 20, -)
```
### トレーリングクロージャー
* 引数で渡す関数が引数リストの最後の引数の場合、次のように、関数名()の後の`{ }`の中に処理内容を記述することができる。
```swift
calculate(10, 20) {
    $0 * $1
}
```
* 引数が関数のみの場合は、関数名の後の`()`も不要。
```swift
func sayHello(greeting: (String) -> String) -> () {
    print(greeting("Hello"))
}
sayHello { $0 + ", World" } // Hello, World
sayHello { "Hi, " + $0 }    // Hi, Hello
```
### 複数のクロージャを渡す際の省略記法
```swift
subject.sink(
    receiveCompletion: { completion in
        print("complete")
    },
    receiveValue: { string in
        print("\(string)")
    }
)
```
下記のように書ける。
```swift
subject.sink { completion in
    print("complete")
} receiveValue: { string in
    print("\(string)")
}
```
### イニシャライザでも同様
* 例えばButton
```swift
init(action: () -> Void, label: () -> Label)
```
* トレイリングクロージャ未使用
```swift
Button(action: { self.number += 1 }, label: { Text("カウント") })
```
* トレイリングクロージャを使用
```swift
Button(action: { () -> Void in self.number += 1 }) {
    Text("カウント")
}
```
