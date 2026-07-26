---
title: "宣言 - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > 宣言


# 代入可能性
https://go.dev/ref/spec#Assignability

# 変数宣言
https://go-tour-jp.appspot.com/basics/12
```go
var i int
var f float64
var b bool
var s string
```
* 変数に初期値を与えずに宣言すると、ゼロ値( zero value )が与えられる。
* ゼロ値は型によって以下のように与えられます:
  * 数値型(int,floatなど): 0
  * bool型: false
  * string型: "" (空文字列( empty string ))
## 型推論
* 宣言時に右辺から型推論される。
  * https://go-tour-jp.appspot.com/basics/14


# 変数宣言のショートハンド
* hoge := 1
  * これはvar hoge = 1 のショートハンド
* a, b := 3, 4
  * これは var a, b = 3, 4 のショートハンド
* したがって以下はエラーになる。
```go
a := 1
a := 2 //再宣言としてエラーになる。
```
* しかし、これの場合はならない。（よしなに、 a = 2　と b := 3に分けてくれている？？）
```go
a := 1
a,b := 2,3
```
## その他ショートハンド
* https://stackoverflow.com/questions/45086082/multiple-variables-of-different-types-in-one-line-in-go-without-short-variable





# 定数
* https://go.dev/ref/spec#Constants
* https://go-tour-jp.appspot.com/basics/15
* const a int = 3
* const a = 3
* 文字(character)、文字列(string)、boolean、数値(numeric)のみで使える。
* structリテラルでは使えない。
* := は使えない。
## 定数宣言ってあんまり使わない？
* 割りと、Goのライブラリでもconstを使って共通の変数を宣言している箇所が少ない。
* というのも、プリミティブだけしかconstが使えないため、使える場所が少ないから。
* (IMO) プリミティブだけconstを使ってもコード上の一貫性がないため、あまり積極的には使わない。
## iota
* 連続した整数定数の表現に利用できるGo独自の識別子
* https://go.dev/ref/spec#Iota
```go
const (
    zero  = iota // 0
    one/* = iota // 1 (省略可)*/
    two/* = iota // 2 (省略可)*/
    three/* = iota //3 (省略可)*/
)
```
```go
type HttpRequestType int

const (
	_ HttpRequestType = iota
	HttpTypeGet
	HttpTypePost
)
func HttpDo(url string, requestType HttpRequestType, token string, body string) (string, error) {
  ...
}
```
* https://qiita.com/curepine/items/2ae2f6504f0d28016411

