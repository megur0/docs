---
title: "struct - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > struct



# 宣言
* https://go-tour-jp.appspot.com/moretypes/2
  ```go
  type Vertex struct {
    X int
    Y int
  }
  func main() {
    v := Vertex{1, 2}// 構造体は、初期化時にフィールド名を省略することができる。
    p := &v
    (*p).X  = 1e9 // この書き方は面倒らしいので、GOではp.Xと書くこともできるとのこと。（なお、C言語みたいにvertex->xって記法は使えない。）
    fmt.Println(v)
  }
  ```
* デフォルト値
  * 構造体のデフォルト値はリテラルの型によって決まっている。
  * intなら0、stringなら""
  * ポインタはnilになる。


# empty struct
* struct{}のこと。
* chanで空で送りたい時に、 struct{}{}みたいに使ったりする。
* メモリサイズが0
* 属性を持たない
* 同じアドレスを示す
* https://devlights.hatenablog.com/entry/2019/12/03/114645



# ポインター
```go
type A struct {
	A string
	B int
}
func main() {
	a := A{"aaa", 3}
	fmt.Printf("%v %v %v", &a, &a.A, &a.B)// &{aaa 3} 0xc000010018 0xc000010028
}
```

# ネスト
* ネストした構造体の初期化
  * 結構癖がある。
  * https://qiita.com/kiida/items/e465ad268bbacf529432
  * https://joniy-joniy.hatenablog.com/entry/2016/12/09/163117


# リテラルのショートハンド
* 下記のように書ける。

```go
  s := []struct {
      i int
      b bool
    }{
      {2, true},
      {3, false},
      {5, true},
      {7, true},
      {11, false},
      {13, true},
    }
  /* 以下のショートハンド（構造体を宣言して、その構造体のsliceを作成）
  type XXXX struct {
    i int
    b bool
  }
  s := []XXXX {
      {2, true},
      {3, false},
      {5, true},
      {7, true},
      {11, false},
      {13, true},
    }
  */
  ```
* type宣言してしまったほうがきれいなケースも多い。下記の場合、ちょっと不細工で、type宣言した方がベターだと思う。
```go
pendingMemberIdAndUserId := util.Map(pendingMembers, func(p repository.PendingMember) struct {
			pendingMemberId string
			userId          string
		} {
			return struct {
				pendingMemberId string
				userId          string
			}{
				p.ID, p.UserId,
			}
		})
...
pendingMemberIdAndUserId = append(pendingMemberIdAndUserId, struct {
				pendingMemberId string
				userId          string
			}{
				existPendingMember.ID, existPendingMember.UserId,
			})
```





# シャローコピー、ディープコピー
* プリミティブだけを含む構造体であれば、ポインタのコピーでディープコピーができる。
```go
t := &T{1, "aaa"}
	a := *t
	a.A = 2
	fmt.Printf("%v\n%v", *t, a) // {1 aaa} {2 aaa}
```
* ネストしててもちゃんとコピーされる。
```go
package main
import "fmt"
type a struct {
  a string
	b string
}
type b struct {
	c string
	d a
}
func main() {
	aa := &b{}
	fmt.Println(aa)//&{ { }}
	bb := &b{}
	*bb = *aa 
	bb.c = "test"
	bb.d.a = "test2"
	fmt.Println(bb)//&{test {test2 }}
	fmt.Println(aa)//&{ { }}
}
```
* https://moneyforward-dev.jp/entry/2021/12/22/go-deep-copy/

# 複雑な構造体のディープコピー
* https://moneyforward-dev.jp/entry/2021/12/22/go-deep-copy/
* https://zenn.dev/tutuz/articles/1826b8383f8d79cd3af3
* https://qiita.com/kz23szk/items/8d60a4716f7e2de2e946
* ポインタや、slice、mapなどの参照型が含まれているものDeepCopyするのは少し骨が折れる。
* ディープコピーのやり方としては、
  * ①jsonなどへのシリアライズ+デシリアライズを使う方法
  * ②リフレクションを使う方法
  * ③deep copyメソッドの自動生成　などがある。
* ただ、①②に関しては速度面がネックではある。（テストコードだったらOK）

# レスポンスで、構造体 -> 他の構造体へコピーしたい箇所をどうするか？
* レスポンスを返すときにモデル -> レスポンスと変換のためにコードが増えるところを削減したい。
* これは上記の①や②で可能ではあるのだが、基本的に冗長な処理なのでコード削減のためにやるべきではないかな、という判断をした。
* もし、go generateとかで省略できるようなものがあれば、いつか使いたい。
* https://zenn.dev/syo_yamamoto/articles/9ddb767dfb85c6


# embbed
* Goでは構造体に型を埋め込むことができる。
* 埋込した型はフィールドとして追加される。
* メソッドの移譲: 埋込した型が持つメソッドを、保持していることになる
	* メソッドはオーバーライドできる
* 埋め込みはサブタイプ化ではない。
	* ただし、埋込したことでinterfaceを満たすことで、そのinterfaceへの代入可能とすることができる。
	* 参考
		* 他のOOP(クラス志向)の言語のextendsやimplemntsのように自身の宣言だけでは代入可能とすることができない。
		* 例えば、AというクラスのモックBを作成したい場合、Goではinterfaceが必要となる。
			* OOPの言語: B extends A として必要なメソッドをオーバーライドする
			* Goの場合: Aの持つメソッドを持つinterfaceを定義。Bにはinterfaceを埋め込み、必要なメソッドをオーバーライドする
```go
package main

import "fmt"

func main() {
	fmt.Println(S2{}.M1())
	fmt.Println(S2{}.M2())
	fmt.Printf("%+v", S2{})

	//var a S1 = S2{} // 埋め込みをしてサブタイプになるわけではない。

	/*

	m1
	m2 overrided
	{S1:{} int:0 Mt:}% // 埋め込みをすると埋込したフィールドが追加される

	*/
}

type S1 struct {
}

func (s S1) M1() string {
	return "m1"
}

func (s S1) M2() string {
	return "m2"
}

type S2 struct {
	S1
	int
	Mt
}

type Mt string

func (s S2) M2() string {
	return "m2 overrided" //オーバーライド可能
}
```
```go
package main

import (
	"fmt"
)

type i1 interface {
	fu() string
}

type s1 struct {
	f1 string
}

func (s s1) fu() string {
	return s.f1
}

type s2 struct {
	f1 string
}

func (s s2) fu() string {
	return s.f1
}

func f1(a1 i1) {
}

type s3 struct {
	s1
	s2
	i1
	f1 string
}

func main() {
	v := s3{f1: "3", s1: s1{f1: "1"}, s2: s2{f1: "2"}}
	fmt.Printf("%#v \n", v)       // main.s3{s1:main.s1{f1:"1"}, s2:main.s2{f1:"2"}, i1:main.i1(nil), f1:"3"} 
	fmt.Printf("%#v \n", v.s1.f1) // 1
	fmt.Printf("%#v \n", v.s2.f1) // 2
	fmt.Printf("%#v \n", v.f1)    // 3
	fmt.Printf("%#v \n", v.s1.fu())// 1
	fmt.Printf("%#v \n", v.s2.fu())// 2
	//fmt.Printf("%#v \n", v.fu()) // ambiguous selector v.fu
    // f1(v): cannot use v (variable of type s3) as i1 value in argument to f1: s3 does not implement i1 (ambiguous selector s3.fu)
}
```

```go
type i1 interface {
	fu() string
}
type s1 struct {
	f1 string
}
func f1(a1 i1) {}
type s3 struct {
	s1
	f1 string
}
func main() {
	v := s3{}
	fmt.Printf("%#v \n", v) 
	f1(v)// embbedすることでinterfaceを実装したことになる。
}
```
```go
package main

import "fmt"

type Test interface {
	A(a string)
}

type S struct {
	Test //s.Test.A("")でもアクセスできるし、s.A("")もアクセスできる。
}

func (s *S) A(a string) {
	fmt.Println("s:" + a)
}

type T struct {
}

func (t *T) A(a string) {
	fmt.Println("t:" + a)
}

func main() {
	s := &S{}
	s.A("test")
	// s.Test.A("test") //panic

	s.Test = &T{}
	s.A("test")
	s.Test.A("test")
}
```
## interface へ 埋め込み?? 
* interfaceの中へ型を含めるものは、構造体のembbedingとは全く異なる意味のため注意。
* これは埋め込みではなく、型制約。
```go
type i interface {
	int
}

func f(a i) {// error: cannot use type i outside a type constraint: interface contains type constraints

}
```
## 埋め込んだ構造体が同じ同名フィールドを保つ場合
* ネストが深いほうが無視される。
* ネストが同じ場合は参照時にエラーになる。
* https://techblog.kayac.com/go-embedding-structs-including-common-keys
## (参考)エラーの実装
* ErrUnexpectedProcess, ErrExpectedProcessを共通化について、下記のように埋め込みしたことで省略できた。
  * newErrUnexpectedProcess, newErrExpectedProcessは無くても良いのだが、
   newErrExpectedProcess{processError("error message"),} のように毎回書くのは結構分かりづらいので。
```go
type processError string
func (e processError) Error() string {
	return string(e)
}

type ErrUnexpectedProcess struct{
	processError
}

func newErrUnexpectedProcess(message string) ErrUnexpectedProcess {
	return ErrUnexpectedProcess{
		processError(message),
	}
}

type ErrExpectedProcess struct{
	processError
}

func newErrExpectedProcess(message string) ErrExpectedProcess {
	return ErrExpectedProcess{
		processError(message),
	}
}
```





# サンプルコード
* 埋め込みを利用してレスポンスを定義した例
```go
package main

import (
	"encoding/json"
	"fmt"
)

type ErrorData struct {
	Message string `json:"message"`
}

type response[T any] struct {
	Result string    `json:"result"`
	Error  ErrorData `json:"error"`
	Data   []T       `json:"data"`
}

func (r *response[T]) Set(result string, err string, data []T) {
	r.Result = result
	r.Error = ErrorData{
		Message: err,
	}
	r.Data = data
}

type MyResponse struct {
	response[any]
}

type MyResponse2 struct {
	response[string]
}

type MyResponse3 struct {
	response[int]
}

// メソッドのオーバーライド(のようなこと)も可能
func (r *MyResponse3) Set(result string, err string, data []int) {
	r.Result = result
	r.Error = ErrorData{
		Message: err,
	}
	r.Data = []int{500}
}

func main() {
	// 埋め込んだresponseがinterfaceのResponseIFを満たしているため、responseを埋め込んだ複数の型を利用できる。
	setHttpResponse(&MyResponse{}, []any{struct{ ID string }{ID: "my id"}})
	setHttpResponse(&MyResponse2{}, []string{"aaa", "bbb", "ccc"})
	setHttpResponse(&MyResponse3{}, []int{})

	// 埋め込みはサブタイプ化ではないため、response型として使うことはできない。
	// setHttpResponse2[string](MyResponse2{}, []string{"aaa", "bbb", "ccc"})
}

type ResponseIF[U any] interface {
	Set(result string, errMessage string, data []U)
}

func setHttpResponse[DATA any, RES ResponseIF[DATA]](res RES, data []DATA) {
	res.Set("success", "", data)
	j, _ := json.Marshal(res)
	fmt.Println(string(j))
}

func setHttpResponse2[DATA any](res response[DATA], data []DATA) {
	res.Set("success", "", data)
	j, _ := json.Marshal(res)
	fmt.Println(string(j))
}

```