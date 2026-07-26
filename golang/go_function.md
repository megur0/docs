---
title: "関数・レシーバー - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > 関数・レシーバー



# 関数
* 関数の引数を　x int, y int　　は x,y int　と書ける。

# クロージャ
* Goの関数はクロージャである。
  ```go
  package main
  import "fmt"
  func main() {
    test := "aaaa"
    fmt.Println(test) //aaaa
    b := func() {
      test := "bbbb"    // :=ではなく=にすると外部のtestを変更する。
      fmt.Println(test) //bbbb
    }
    b()
    fmt.Println(test) //aaaa
  }
  ```
* Goの関数は第一級関数であり、高階関数が使える。
  * https://go-tour-jp.appspot.com/moretypes/24
  * 関数内で関数が扱える
  ```go
  func a() {
    b := func() {}
    b()
    //func c() {} //ただし、これはシンタックスエラーとなる。
    //c()
  }
  ```
* 参考
  * https://code-graffiti.com/closure-in-golang/
  * (?) ginのrouter groupでクロージャらしきものを使っているが、使っている理由がよく分からない。（単にコードを見やすくするためか？）
    * https://korattablog.com/2020/07/18/gin%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%9Fgo-api%E9%96%8B%E7%99%BA%E3%81%AE%E5%88%9D%E6%AD%A9%EF%BC%88%E3%83%AB%E3%83%BC%E3%82%BF%E3%83%BC%E3%82%B0%E3%83%AB%E3%83%BC%E3%83%97%E5%8C%96%E7%B7%A8%EF%BC%89/
  * https://blog.y-yuki.net/entry/2017/05/04/000000


# variadic functions（可変長引数関数）
* 可変長引数は配列として受け取る。
```go
func Query(db *sql.DB, args ...any) {
  ...
  if len(args) ...
}
```
* 配列は可変長引数として展開できる。
```go
opts := []cmp.Option{
  cmpopts.IgnoreFields(entity.User{}, "", ""),
}
if diff := cmp.Diff(want, respObj, opts...); diff != "" {// 配列を可変長引数として展開
  t.Errorf("Compare value is mismatch (-v1 +v2):%s\n", diff)
}
```

# multiple return values
* 日本語だと、「多値」？
* https://go.dev/tour/basics/6
* 戻り値は分配できる
  *  x, y := pair(10)
* 関数の戻り値は複数であっても、そのまま他の関数に渡せる。
  * https://go.dev/ref/spec#Calls
  ```go
  func Split(s string, pos int) (string, string) {
	  return s[0:pos], s[pos:]
  }
  func Join(s, t string) string {
    return s + t
  }
  if Join(Split(value, len(value)/2)) != value {
    log.Panic("test fails")
  }
  ```

# Named return values
* ただ、若干可読性が悪い気もする。（短い関数であれば良いかも。）
```go
func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}
```
* https://go-tour-jp.appspot.com/basics/7


# optional parameter, default value, named parameter
* いずれもできない。
* (IMO) named parameterは構造体を渡すことで擬似的に実現できるが、記述が長くなるためあまり好ましいとは思わない。


# defer
* 実行を関数の終わりまで遅延させる。
  * https://go-tour-jp.appspot.com/flowcontrol/12
* deferが発火するのは、関数呼び出しから戻る時のみ。
```go
import "fmt"

func main() {
	f(func() {
		defer fmt.Println("callback end")
		fmt.Println("callback")
	})
}

func f(a func()) {
	a()
	fmt.Println("f end")
}

/*
callback
callback end
f end
*/
```
## deferはos.exitが呼ばれると意図しない挙動になる？
* https://budougumi0617.github.io/2021/06/30/which_termination_method_should_choose_on_go/
## panicとの関係
* panicのときもdeferは実行される。
* 詳細は[エラーハンドリング](./go_error.md)を参照
## defer 関数 とブロックスコープ
* defer は関数呼び出しから戻る際に実行されるのであって、ブロックスコープの末尾ではない。
* 下記の感じだと、後に呼んだdeferが先に実行される感じ？
* 
```go
func main() {
	defer println("5")
	{
		defer println("4") // 囲んでいるブロックスコープの末尾ではなく、関数（main関数）の最後に実行される。
		println("1")
	}
	defer println("3")
	println("2")
}
/// 1 -> 2 -> 3 -> 4 -> 5
```


# 特殊な関数　init 
* https://golang.org/doc/effective_go#init
* init、変数の初期化の順番
  * https://qiita.com/YusukeIwaki/items/f1f92c23d7ee0ca8dc7a
  * 変数の初期化のほうが先に実行される。
  * 各モジュールのinitはそれぞれの順番は保証できないので、順番に依存しないように注意しないとね。
  * blankインポートを一緒に使うと便利かも。
## 初期化のパターン
1. varで初期化する。
```go
var (
	// std is the name of the standard logger in stdlib `log`
	std = New()
)
```
2. Init関数で初期化する。
3. Init関数を使わずに、明示的に自分で関数を作ってmain等から呼ぶ。
## var, init, main の順番
* https://qiita.com/suin/items/ab2db295742afcf02334
1. importしたパッケージのvarが定義される
2. importしたパッケージのinit関数が実行される
3. mainパッケージのvarが定義される
4. mainパッケージのinit関数が実行される
5. mainパッケージのmain関数が実行される



# レシーバー
* メソッド
  * 任意の型（type）に対してメソッドを定義することができる。
  * メソッド: 関数において、タイプに所属する関数
  * レシーバー: メソッドに型が指定されている箇所
  ```go
  func (v Vertex) Abs() float64 { /// <- この  (v Vertex) で指定している箇所をレシーバーという
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
  }
  func main() {
    v := Vertex{3, 4}
    fmt.Println(v.Abs())
  }
  ```
* ポインタレシーバー/値レシーバー
  * ポインタレシーバーにポインタではなく変数を渡した場合は自動的にポインタに変換される。
  * 値レシーバーにポインタを渡した場合は自動的に参照先の値に変換される。
  * https://go-tour-jp.appspot.com/methods/1
  * ポインタレシーバーを使うメリット
    * 指す先を変更するため。
    * 大きな構造の場合にメソッド側でコピーを作らない
  * 一般的に値レシーバーとポインタレシーバーを混在させるべきではない。
  * まあ、個人的にポインタレシーバー統一で良いかな。
    * https://kazuhira-r.hatenablog.com/entry/2021/01/04/131425
  * でも、そもそも、後述しているけど、値レシーバーは自身を変更することができないから、
    * 混在するのは間違っていないと思う。
    * そもそもjson.Marshalerやjson.Unmarshalerも、値レシーバーとポインタレシーバーだし。
## メソッドと関数は本質的に同義
```go
func (p Person) Greet(msg string) {
    // ...
}
// 実は、これは下記と等価らしい。
func Person.Greet(p Person, msg string) {
    // ...
}
```
* https://skatsuta.github.io/2015/12/29/value-receiver-pointer-receiver/
* https://zenn.dev/spiegel/articles/20201212-method-value-and-expression
## サンプルコード
```go
import (
	"encoding/json"
	"reflect"
	"testing"
	"time"

	"github.com/stretchr/testify/assert"
)

// func GetFirst[A any](a A, b ...any) A {
// 	return a
// }

// func GetLast(a ...any) any {
// 	return a[len(a)-1]
// }

// func UnmarshalString[S any](jsn string, to *S) (*S, error) {
// 	if err := json.Unmarshal([]byte(jsn), to); err != nil {
// 		return nil, err
// 	}
// 	return to, nil
// }

func Ptr[T any](a T) *T {
	return &a
}


func TestReciever(t *testing.T) {
	t.Run("assert_reciver", func(t *testing.T) {
		assert.Exactly(t, time.Time{}.IsZero(), true)
		assert.Exactly(t, Ptr(time.Time{}).IsZero(), true)
		assert.Exactly(t, Ptr(time.Now()).IsZero(), false)

		assert.Exactly(t, testStruct1("").valueReciever(), true)
		assert.Exactly(t, testStruct1("").pointerReciever(), true)    //値型のレシーバーじゃなくても呼べる。
		assert.Exactly(t, Ptr(testStruct1("")).valueReciever(), true) //ポインタ型のレシーバーじゃなくても呼べる。
		assert.Exactly(t, Ptr(testStruct1("")).pointerReciever(), true)

		// 値レシーバーを実装していると、そのポインターも実装していることになる。
		// しかし、ポインタレシーバーを実装しているときは、その値は実装していることにならない。
		_, ok := reflect.ValueOf(time.Time{}).Interface().(json.Marshaler)
		assert.Exactly(t, ok, true)
		_, ok = reflect.ValueOf(&time.Time{}).Interface().(json.Marshaler)
		assert.Exactly(t, ok, true)
		_, ok = reflect.ValueOf(time.Time{}).Interface().(json.Unmarshaler)
		assert.Exactly(t, ok, false) // これはfalseになる
		_, ok = reflect.ValueOf(&time.Time{}).Interface().(json.Unmarshaler)
		assert.Exactly(t, ok, true)
	})
}

type testStruct1 string

func (t testStruct1) valueReciever() bool {
	return true
}
func (t testStruct1) pointerReciever() bool {
	return true
}
```

## 値レシーバーは自身を変更することができない？
```go
type s1 struct {
	f1 string
}

func (s s1) set() {
	s.f1 = "aaaa"
}

func (s *s1) set2() {
	s.f1 = "bbbb"
}

func main() {
	v1 := s1{f1: "0000"}
	v1.set()
	fmt.Println(v1.f1)//0000
	(&v1).set2()
	fmt.Println(v1.f1)//bbbb
}
```
## (参考)レシーバーがポインターと値の２種類それぞれに対して処理を変える必要がある
* レシーバーが、ポインターレシーバーと値レシーバーの２択あるため、ライブラリの実装に応じて処理を変えなければならない。
* 例えば、errors.Asの第二引数は、interface{Error()string}を満たす型へのポインタを渡す必要がある。
* この場合、値レシーバーで実装されている場合はその値へのポインタを渡す
* 一方、ポインタレシーバーで実装されている場合はそのポインタへのポインタを渡す。
* 例えば、json系のパッケージのエラーはポインタレシーバーで実装されているため、ポインタのポインタを渡す必要がある。
```go
p := &json.UnmarshalTypeError{}
fmt.Println(errors.As(json.Unmarshal([]byte("{"), &struct{}{}), &p)) 
```
* https://stackoverflow.com/questions/69447919/panic-errors-target-must-be-interface-or-implement-error-in-go




# 引数として渡した場合の元の値の変更
* Goでは関数へ渡す際はすべて、（いわゆる）値渡しとなる。
  * https://go.dev/ref/spec#Calls
  > In a function call, the function value and arguments are evaluated in the usual order. After they are evaluated, the parameters of the call are passed by value to the function and the called function begins execution. The return parameters of the function are passed by value back to the caller when the function returns.
* したがって、その値自体を関数側で変更したとしても、元の値は変わらない。
* 一方、ポインターやmapやスライスはそれ自体がデータへの参照となるため、関数側でその参照先を取得することができる。
* コードで確認したところ、スライスは参照だが、関数側で参照先を変えても元の参照先は変わらなかった。
```go
package main

import "fmt"

func main() {
  // スライスは参照だが、関数へ渡したスライスの参照先を変更すると、ポインタ自体が置き換わり、元の参照先は変わらない仕様となっている。
	a := []string{}
	fmt.Printf("before call: %p\n", a)
	fmt.Println(a)
	f(a)
	fmt.Printf("after call: %p\n", a)
	fmt.Println(a)
	fmt.Println("_______")

  // mapは参照であり、関数へ渡したmapの参照先を変更すると、元の参照先も変わる。
	b := map[string]string{}
	fmt.Printf("before call: %p\n", a)
	fmt.Println(b)
	f2(b)
	fmt.Printf("after call: %p\n", a)
	fmt.Println(b)
	fmt.Println("_______")

  // 配列の場合は参照でないため、関数側に渡した値を変更しても元の値は変わらない。
	c := [3]string{"a", "b", "c"}
	fmt.Println(c)
	f3(c)
	fmt.Println(c)
	fmt.Println("_______")
}

func f(a []string) {
	fmt.Printf("before append %p\n", a)
	a = append(a, "a")
	fmt.Printf("after append %p\n", a)
}

func f2(a map[string]string) {
	fmt.Printf("before append %p\n", a)
	a["a"] = "a"
	fmt.Printf("after append %p\n", a)
}

func f3(a [3]string) {
	a[2] = "a"
}

/*

before call: 0x590360
[]
before append 0x590360
after append 0xc000014070
after call: 0x590360
[]
_______
before call: 0x590360
map[]
before append 0xc0000160f0
after append 0xc0000160f0
after call: 0x590360
map[a:a]
_______
[a b c]
[a b c]
_______

*/
```
