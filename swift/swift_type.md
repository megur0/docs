---
title: "型（struct・class・enum） - Swift"
updated: 2026-07-26
---

[TOP(About this memo))](../README.md) > [一覧(Swift)](./README.md) > 型（struct・class・enum）


# 値型と参照型
* (参考) https://swift.codelly.dev/guide/%E6%A7%8B%E9%80%A0%E4%BD%93%E3%81%A8%E3%82%AF%E3%83%A9%E3%82%B9/%E5%80%A4%E5%9E%8B%E3%81%A8%E5%8F%82%E7%85%A7%E5%9E%8B.html#%E5%80%A4%E5%9E%8B
* 構造体と列挙型は値型で、クラスは参照型。
* Swiftでは、基本的な型であるInt型、Float型、Bool型、String型、Array型、Dictionary型などは全て値型。
    * これらは内部では構造体として実装されている。
    * TIP: コレクションはコピーする際のパフォーマンスコストを削減するために最適化が行われている。すぐにコピーを作るのではなく、元のインスタンスとコピー先のインスタンスの間で値のメモリを共有し、コピー先のコレクションの要素が1つでも変更された場合に値がコピーされる（Copy-on-Write）。
* クラスは、`===` と `!==` で同じインスタンスかどうかを調べることができる。
* `===` は、クラス型の変数が同じインスタンスを参照していることを意味する。
* `==` は、2つのインスタンスが等しい、または値が同等であることを意味する。
* ObjectIdentifierを使うと、クラスの一意の識別子を調べることができる。
* Swiftでは、なるべく値型を使い、参照型は必要最低限にすると良い。


# インスタンス
* 構造体もクラスもenumも同様に「インスタンス化」が可能。


# イニシャライザ
* initは型のコンストラクタ。
```swift
let myString = String.init("Hello, World!")
```
普通は省略形として下記のように書く。
```swift
let myString = String("Hello, World!")
```
型が決まっている場合は`.init`で省略表記可能。
```swift
let a: String = .init("aaaaa")
```
* 関数の戻り値も型を省略して書ける。
```swift
private func isActiveDestination(_ number: Int) -> Binding<Bool> {
    .init(
        get: { selectedNumber == number },
        set: { selectedNumber = $0 ? number : nil }
    )
    /*
    上記は下記のようにも書ける。
    return Binding(
        get: { selectedNumber == number },
        set: { selectedNumber = $0 ? number : nil }
    )
    */
}
```


# プロパティ
* クラスや構造体、列挙型で定義することができる定数/変数。
* 列挙型では、コンピューテッドプロパティおよびタイプ（型）プロパティのみ使える（ストアドプロパティおよびインスタンスプロパティは使えない）。
    * 列挙型はそもそも、予め決まった値にもとづいたものなので。
    * (参考) https://qiita.com/hachinobu/items/392c96820588d1c03b0c
* 格納するかどうか
    * ストアドプロパティ
        * イニシャライザ(init)
            * すべてのストアドプロパティはインスタンス化時に値が入っている必要がある。
                * したがってインスタンス化時に引数で入れていないとコンパイルエラーになる。
                * ただし、ストアドプロパティに初期値が与えられている場合は省略できる。
            * クラスも構造体もinit（コンストラクタ的なもの）でストアドプロパティを初期化できる（enumもinitは使えるが、ストアドプロパティではなく自身の値を設定する）。
            * イニシャライザ（init）を使用する場合、定義しているストアドプロパティ全てに値を格納しないといけない。
                * ただし、初期値が与えられているストアドプロパティは省略できる。
            * 構造体はinitが定義されていない場合は暗黙のinitが定義される。
            * クラスの場合は暗黙のinitはない（はず）。したがって、initがない、かつ初期値を与えていないストアドプロパティがある場合はコンパイルエラーになる。
        * プロパティオブザーバ
            * 格納型プロパティの値が更新される時、それをきっかけに処理を書くことができる。
            * willSet
                * プロパティの値が更新される直前に呼び出される。
            * didSet
                * プロパティの値が更新される直後に呼び出される。
    * コンピューテッドプロパティ
        * Swift 5.1からは式がひとつであればreturnを省略可能。
* インスタンス化が必要かどうか
    * インスタンスプロパティ
        * インスタンス化しないと参照できないプロパティ。
    * タイプ（型）プロパティ
        * インスタンス化しなくても参照できるプロパティ。
        * staticキーワードを使用して型プロパティを定義する。
        * クラス型の計算型プロパティの場合、代わりにclassキーワードを使用して、サブクラスがスーパークラスの実装をオーバーライドできるようにすることもできる。


# struct
* 構造体の中には定数、変数、イニシャライザ、関数が定義可能。
* 構造体にはプロトコルを指定することもできる。
```swift
struct 構造体名: プロトコル {
    let name: String // ストアドプロパティ(stored property)
    var age: Int {   // コンピューテッドプロパティ(computed property)
        return 20 // Swift 5.1からは式がひとつならreturnを省略可。
    }
    init(name: String) {
        self.name = name // ageの初期化は不要
    }
    func メソッド() -> 戻り値の型 {
        return 戻り値
    }
}
```
* 構造体を変数に格納すると、プロパティやメソッドにはドットシンタックスを使ってアクセスできる。
* 構造体に定義しているメソッドからプロパティの値を変化させようとするとエラーが発生する。
    * これを防ぐにはメソッドの前にmutatingをつける。
* 構造体は継承ができない。
* (参考) https://tech.amefure.com/swift-struct


# mutating
* StructやEnumのような値型では、自身の値を変更する場合、メソッドの宣言にmutatingキーワードをつける必要がある。
* mutatingキーワードが指定されたメソッドの呼び出しは再代入として扱われるので、定数に格納された値型のインスタンスに対しては実行できずコンパイルエラーとなる。
```swift
struct Character {
    var hitPoint: Int = 100
    func damage() { // mutating func damage とする必要がある（なお、SwiftUIなら @State var hitPoint: Int = 100 とするやり方でもOK）
        hitPoint -= 30 // error: Left side of mutating operator isn't mutable: 'self' is immutable
    }
}
```


# クラス
* init（initializer）
* Swiftはシングル継承モデルを使用しており、どのクラスも1つのスーパークラスしか持たない。
* Swiftのクラスは複数のインターフェイス（プロトコルとも呼ばれる）を実装することができる。
* self
* アクセス制御
    * open、public、internal、file-private、private（Dartはpublicとprivateのみ）。
* 計算型プロパティ (Computed property)
    * インスタンスが生成される度に別プロパティから計算されて値を取得、設定されるプロパティでgetter/setterがある。
    * getterは単独で定義できるが、setterはgetterとセットでないと定義できない。
    * getterのみの場合はget{}を省略できる。
    * setter
        * インスタンス化された後、computed propertyに別の値がセットされた時に呼ばれる。
        * getはインスタンス化された時や、setによってgetで使っている値が変更された時に呼び出される。
        * setterは引数のかわりにnewValueという予約語を使うことができる。
    ```swift
    var triangle: Int {
        // 〜〜
        set(t) {
            base = t * 2
            height = t * 2
        }
        // 以下を使える。
        // set {
        //     base = newValue * 2
        //     height = newValue * 2
        // }
        // 〜〜
    }
    ```


# Enum
* (TODO) enum値を定義する練習をしておく。
* メリット
    * 可読性が高い、コード補完で入力できる、switch文で使う際は網羅していないとコンパイルエラーになるので保守性がある。
* Swiftの列挙型は、それ自体が第一級の型。
    * (参考) https://www.swiftlangjp.com/language-guide/enumerations.html
* あらかじめ型が宣言されている場合は、値を入れるときに型を省略できる。
```swift
let myPet = Animal.dog
```
は以下のように書ける
```swift
var myPet: Animal
myPet = .dog
```
* initで初期値設定も可能。
* なお、initを定義していない場合は自動的に下記のようなinitが定義される。
    * (参考) https://zenn.dev/ikuraikura/articles/2022-10-18-enum
```swift
init?(rawValue: String) {
    switch rawValue {
    case 〜:
        self = .〜
    case 〜:
        self = .〜
    default:
        return nil
    }
}
```
* enumはネストして定義できる。`let myPet = Animal.Dog.chihuahua`
* 各caseに付属値を設定できる。
* メソッドが設定できる。
* 算出型プロパティとタイププロパティが使える（格納型プロパティは使えない）。
* プロトコルに準拠させることができる。
* (参考) https://shuhey-hashimoto.com/swift/【swift】列挙型の使い方/


# デコード・エンコード(WIP)
* Decodable
