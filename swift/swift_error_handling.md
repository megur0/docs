---
title: "エラーハンドリング - Swift"
updated: 2026-07-26
---

[TOP(About this memo))](../README.md) > [一覧(Swift)](./README.md) > エラーハンドリング


## エラーを投げる（throws, throw）
```swift
func メソッド名(引数名: 型) throws -> 戻り値 {
    // エラーを投げる可能性のある処理を記述
    throw エラーを投げる処理内容
}
```


## do try catch
```swift
do {
    try 〜
} catch CocoaError.fileNoSuchFile {
    // ファイルやディレクトリが見つからなかった場合の処理
    // この書き方の場合は、自動的に変数errorがError型として値が入っている。
} catch CocoaError.fileLocking {
    // ファイルがロックされている場合の処理
} catch {
    // その他のエラー時
}
```
* エラーのカテゴリ単位で処理したいとき
```swift
} catch let error as CocoaError {
    // CocoaErrorだったときの共通処理
    doSomething()

    switch error.code {
    case .fileNoSuchFile:
        // ファイルやディレクトリが見つからなかった場合の処理
    case .fileLocking:
        // ファイルがロックされている場合の処理
    default:
        // その他のCocoaError時
    }
} catch let error as 〜 {
} catch {
    // その他のエラー時
}
```
* カスタムエラーで値を受け取るenumを定義している場合
```swift
enum CustomError: Error {
    case something(message: String)
}
do {
    throw CustomError.something(message: "何かのエラーです")
} catch CustomError.something(let message) {
    print(message)
} catch {
    // 何かエラー処理
}
```
* try? try!
```swift
try? FileManager.default.removeItem(atPath: path)
try! FileManager.default.removeItem(atPath: path)
```
* defer
    * Swiftではfinallyは無い。
    * その代替がdefer。
    ```swift
    do {
        defer {
            fileReader.close()
        }
        try 〜
    }
    ```
    * 基本的にtryを行う前にdefer文を書く。言語仕様上、もしdeferの前にtryでエラーが起きると、deferが評価される前に例外がthrowされてしまうため。
    * defer文は何度も書ける。defer文は定義されたのが「後」のほうが先に実行される。
* (参考) https://dev.classmethod.jp/articles/about-error-handling/
* Errorプロトコル
    * Errorプロトコルに準拠しているものは何でもthrowできる。
    * わかりやすくするために、エラーはenumで定義しておき、Errorに準拠させるのがベスト。
    * さらに、ErrorではなくLocalizedErrorに準拠させることで、MyAppErrorから直接`.errorDescription`にアクセスできる。
    * enumをネストして使うことで、より構造化しやすい。
    * (参考) https://zenn.dev/kyome/articles/a2323f7eb2bf92
