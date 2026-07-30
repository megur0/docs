---
title: "ポインタ - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > ポインタ





## 公式(A tour of Go)
* https://go-tour-jp.appspot.com/moretypes/1

## new
* 指定したポインタ型を生成
```go
type person struct {
    height float32
    weight float32
}
...
p := new(person)
```
* genericsにも使える。
```go
func DeepCopy[T any](src *T) *T {
	dst := new(T)
    ...
}
```


## プリミティブ
* プリミティブのリテラルや関数の戻り値から直接アドレスを取得することはできない。
```go
package main

import "fmt"

type myString string

type myStruct struct{}

func main() {
	// fmt.Println(&3) // error
	// fmt.Println(&f())// error
	// fmt.Println(myString("aaa")) //error
	fmt.Println(&struct{ a string }{a: "aaa"}) // 構造体は可能
	fmt.Println(&myStruct{})

	a := 3
	fmt.Println(&a)

	b := f()
	fmt.Println(&b)
}

func f() string {
	return "fff"
}
```
* 下記のように関数を定義することで1行で書くことが可能
```go
// &3 -> Ptr(3)
// &SomeFunc()-> Ptr(SomeFunc())
func Ptr[T any](a T) *T {
	return &a
}
```
* https://github.com/golang/go/issues/45653#issuecomment-915602768