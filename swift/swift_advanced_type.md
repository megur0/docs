---
title: "any と some（Existential/Opaque Type） - Swift"
updated: 2026-07-26
---

[TOP(About this memo))](../README.md) > [一覧(Swift)](./README.md) > any と some（Existential/Opaque Type）


# any と some
## Existential Type
* (参考) https://zenn.dev/ueshun/scraps/27348037af9e0b
* any
* そもそも、プロトコルを型として使って実行時に実際の型が決まるようにしてしまうと、メモリ使用量が増える。
* こういうケース
```swift
let animal: Animal = Bool.random() ? dog : cat
```
* そのため、Swift 5.6からはanyというキーワードが追加され、「任意のxxx」という強調された表記になった（理由がない限り、anyはなるべく使わない方が良い、という強調的な意味合い）。
```swift
let animal: any Animal = Bool.random() ? dog : cat
```
* anyはsomeと違って実行時に動的に型が決まる。

## Opaque Result Type
* Swift 5.1から追加。
* some
* プロトコル自体を明示せずに、あるプロトコルに準拠しているということを表す。
* コンパイラ段階で最終的に型が解決するため、実行時のパフォーマンスロスも無い。
* anyと違ってコンパイル時に型が決まるので、下記のようなコードはコンパイルエラーになる。
```swift
func randomAnimal2() -> some Animal {
    Bool.random() ? Dog() : Cat()
}
```

## 具体例
* anyについて、下記の例だとプロトコルのbodyのView型の条件を満たせない（直感に反する挙動）。
```swift
struct MyView: View {
    // var body: View = Text("Hello, World!") // そもそも、anyをつけろと怒られる
    // var body: any View = Text("Hello, World!") // これだと、Viewプロトコルに準拠していないと怒られる（任意のViewだったら満たしそうにも思えるが）。
    var body: some View = Text("Hello, World!") // これだとOK。
}
```
