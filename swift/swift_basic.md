---
title: "基本文法 - Swift"
updated: 2026-07-26
---

[TOP(About this memo))](../README.md) > [一覧(Swift)](./README.md) > 基本文法


# 記法
* 構造体名はパスカル記法(UpperCamelCase)、プロパティやメソッドはローワーキャメル記法(lowerCamelCase)を用いることが推奨されている。


# print
```swift
let range = 0.0...3.5
print("type: \(type(of: range))")
```


# 変数
* var
```swift
var age1: Int
age1 = 44
var age2: Int = 42
var age3 = 43 // 型推論
```
* letはDartでいうfinalで、再代入ができない。
    * JavaScriptのletとは意味が異なるので注意。
* オプショナル、ノンオプショナル
    * (参考) https://qiita.com/maiki055/items/b24378a3707bd35a31a8
    * `var age: Int?` のように書くと、nilを許容するオプショナル型になる。
    * オプショナルではない値として使うにはアンラップが必要。
        * 強制的アンラップ
        ```swift
        var optional: Int? = 10
        print(optional)  // => Optional(10)
        print(optional!) // => 10 強制的アンラップ
        ```
        * 変数の値がnilの場合はエラーとなりアプリが落ちるので注意。
        * オプショナルバインディング
        ```swift
        var hobby: String?
        // 〜〜
        if let unwrappedHobby = hobby {
            // hobbyがnilではない場合
            print(unwrappedHobby)
        } else {
            // hobbyがnilの場合
            print("趣味はありません")
            // => "趣味はありません"
        }
        ```
        * Optional Chaining
            * nilの場合はnilが返る。nilでない場合は値が返るが、戻り値がOptionalになる点に注意。
            ```swift
            オプショナル型の変数?.プロパティ
            オプショナル型の変数?.メソッド()
            ```
    * 暗黙的アンラップ型(Implicitly Unwrapped Optional)
        ```swift
        var a: Int  = 10 // 非オプショナル型
        var b: Int? = 10 // オプショナル型
        a + b! // ! を付ける必要がある
        var a: Int  = 10 // 非オプショナル型
        var b: Int! = 10 // 暗黙的アンラップ型 "!"
        a + b // ! を付ける必要がない
        ```
        * 強制的アンラップと同様に、変数の値がnilの場合はエラーとなりアプリが落ちるので注意。
        * 暗黙的アンラップ型は、最初はnilで宣言したいが、使うときには必ず値が入っているような場合に使用する。


# 範囲型（Range）
```swift
let range2 = 0...3
let range = 0..<3
let range4 = 0.0...3.5
```


# コレクション
* 配列 (Array)、セット (Set)、ディクショナリ (Dictionary)。


# 制御構文
* for
```swift
for num in [10, 20, 30, 40, 50] {
    if num == 30 {
        break
    }
    print(num)
}
```
* インスタンスメソッドを使ってクロージャ式を渡しても同じことができる。
```swift
[10, 20, 30, 40, 50].forEach({ num in /* 〜〜 */ })
[10, 20, 30, 40, 50].forEach { num in /* 〜〜 */ }
```
* guard
```swift
guard let contents = try? FileManager.default.contentsOfDirectory(atPath: directoryPath) else {
    return []
}
return contents
```
