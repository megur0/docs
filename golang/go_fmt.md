---
title: "fmt - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > fmt




## fmt
* https://qiita.com/rock619/items/14eb2b32f189514b5c3c
```go
a := 42
//fmt.Printf("%s\n", a) // error
fmt.Printf("%T\n", a)  // int
fmt.Printf("%v\n", a)  // 42
fmt.Printf("%+v\n", a) // 42
fmt.Printf("%#v\n", a) // 42
fmt.Printf("%b\n", a)  // 101010
fmt.Printf("%c\n", a)  // * // Unicodeコードポイントに対応する文字
fmt.Printf("%d\n", a)  // 42
fmt.Printf("%o\n", a)  // 52
fmt.Printf("%q\n", a)  // '*' //対応する文字をシングルクォート'で囲んだ文字列
fmt.Printf("%x\n", a)  // 2a
fmt.Printf("%X\n", a)  // 2A
fmt.Printf("%U\n", a)  // U+002A

b := 0.2
fmt.Printf("%f\n", b) //0.200000
fmt.Printf("%.60f\n", b) //0.200000000000000011102230246251565404236316680908203125000000

```
```go
a := struct {
  F1 string `json:"f1"`
  F2 int    `json:"f2"`
}{
  F1: "v1",
  F2: 5,
}
j, _ := json.Marshal(&a)
jj, _ := json.MarshalIndent(&a, "", "    ")

fmt.Printf("%s\n", string(j))  // {"f1":"v1","f2":5}
fmt.Printf("%s\n", string(jj)) // 改行、インデントの付いたjson
fmt.Printf("%v\n", a)          // {v1 5}
fmt.Printf("%+v\n", a)         // {F1:v1 F2:5}
fmt.Printf("%#v\n", a)         // struct { F1 string "json:\"f1\""; F2 int "json:\"f2\"" }{F1:"v1", F2:5}
```
* MarshalIndent って関数を使うとインデントとか改行も入れてくれるっぽい。
### 複数
* `fmt.Println(“v1=”, v1, “,v2=”, v2) `


## fmtに定義されるインターフェース
* https://budougumi0617.github.io/2019/10/12/confirm-print-with-fmt-interfaces/