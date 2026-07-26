---
title: "nil - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > nil




# nilリテラルのvariableへの代入
```go
import (
	"fmt"
	"reflect"
)

func main() {
    // 型のないnilのvariableはコンパイルエラーとなる
	// var a = nil // use of untyped nil in variable declaration

    // ポインタ型には格納できる
	var a *int = nil 
	fmt.Printf("%v, %v \n", reflect.TypeOf(a), reflect.ValueOf(a))// *int, <nil> 

    // interface型にも格納できる
    var ii i = nil
	fmt.Printf("%v, %v \n", reflect.TypeOf(ii), reflect.ValueOf(ii))// <nil>, <invalid reflect.Value> 
    var err error = nil
	fmt.Printf("%v, %v \n", reflect.TypeOf(err), reflect.ValueOf(err))// <nil>, <invalid reflect.Value> 
	var iii i = (*int)(nil)
	fmt.Printf("%v, %v \n", reflect.TypeOf(iii), reflect.ValueOf(iii))// *int, <nil> 
}
```

# nilリテラル、nilが格納されたvariable, interface, それらの型情報
* nilリテラル自体は型情報は持っていない。
* 型を持つvariableにnilを格納することで、「値がnilでXXX型の変数」ができる(と思われる)
* これがinterface型に変換されると、interface型の構造には型情報(type)と値(data)として保持される。
    * 例えば上記のreflect.TypeOfやreflect.ValueOfはany型として値を受取り、そのtypeやdataを表示している。

# nilのキャスト
* nilはキャストできる。
```go
package main

func main() {
	f(nil)// ポインターの場合はnilを持つことができるため問題ない。
	//f2(nil)// Cannot infer T 
	f2((*string)(nil))
	f2[any](nil)// なお、これでも通る
}	

func f(a *string) {}
func f2[T any](a T) {}
```

# nilの比較
* nil リテラル同士だと比較はできない
```go
//println(nil == nil) // operator == not defined on untyped nil
```
* キャストをしたnilの比較ができるかどうかは、その型が比較できるかどうかと同様となる。
```go
//println((*int32)(nil) == (*int64)(nil)) //mismatched types *int32 and *int64
println((*int32)(nil) == (*int32)(nil)) // true
println((*int32)(nil) == nil) // これも、true
println(nil == (error)(nil))// これもtrue
```
* reflectionを使った判定
```go
println(reflect.ValueOf((*int32)(nil)).IsNil()) // true
```
## Typed-nil問題　
* 簡単に言うと、interface型において、typed nil（typeはnilではなく、dataがnil）とnil（typeもdataもnil）は等しくないので注意、という話。
* 詳細は[インターフェース](./go_interface.md)を参照







