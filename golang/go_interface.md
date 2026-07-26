---
title: "インターフェース - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > インターフェース





# 仕様
* https://go.dev/ref/spec#Interface_types
> An interface type defines a type set. A variable of interface type can store a value of any type that is in the type set of the interface. Such a type is said to implement the interface. The value of an uninitialized variable of interface type is nil.


# インターフェース型の定義
* 定義
	* /usr/local/go/src/runtime/runtime2.go
```go
type iface struct {
	tab  *itab
	data unsafe.Pointer
}
```
* 概念としてインターフェース値はtypeとdataから構成
* インターフェースのゼロ値はtypeとdataがともにnil


# interface{}
* ゼロ個のメソッドを指定されたインターフェース型。空のインターフェース 
* 下記のように定義されているため anyのキーワードとして扱うことができる
	```go
	type any = interface{}
	```
* 空のインターフェースは、任意の型の値を保持できる。
	```go
	var i interface{} = "hello"

	func f(a any) {}

	func main() {
		f("a")
	}
	```


# intefaceは何とでも比較ができる。
```go
import "fmt"

func main() {
	//f0("aaaa")
	f1("aaaa")
	f1(1)
	f1(nil)
	f1((*int)(nil))
}	

func f0(a int) {
	fmt.Println(a == 1)
}

func f1(a any) {// any型として受け取る
	fmt.Println("---")
	fmt.Println(a == 1)// anyはどの型とも比較ができる
	fmt.Println(a == nil)
	fmt.Println(a == (*int)(nil))
}
```


# インターフェース型の代入可能性
* Goがインターフェースに代入可能となるためにはそのメソッドが実装されているだけでよい。（暗黙的、implicit）
* これは、
	* interface宣言したものは、単にinteface型として、typeとvalueから構成されるものであり、
	* 実装する側は、それとの関連はなくて、型判定するときに構造的に一致しているかどうかを判定するから？？
* https://go-tour-jp.appspot.com/methods/9
* https://go-tour-jp.appspot.com/methods/10
* https://go-tour-jp.appspot.com/methods/11

```go
package main

func main() {
	f1(t1("test"))
	f2(t1("test"))
}

func f1(a i1) {
	t := a.(t1)// interfaceは具象の情報を持つのでアサーションが可能
	println(t.m2())
	println(a.m())
}

func f2[U i1](a U) {
	println(a.m())
}

//  i1 に関する記述は不要 であることが特徴
type t1 string

func (t t1) m() string {
	return string(t)
}

func (t t1) m2() string {
	return string(t) + string(t)
}

// interface
type i1 interface{ m() string }

```
## nilはinterface型に代入可能
* nil は interface型に代入可能である。
## interface型からinterface型への代入可能性
* interface同士でもinterfaceを満たしていれば代入可能。
```go
type I interface{}
type I2 interface{ M() }
type I3 interface {
	M()
	N()
}

func main() {
	var i I
	var i2 I2
	var i3 I3

	i = i2
	i = i3
	// i2 = i // annot use i (variable of type I) as I2 value in assignment: I does not implement I2 (missing method M)
	i2 = i3
	// i3 = i2 //cannot use i2 (variable of type I2) as I3 value in assignment: I2 does not implement I3 (missing method N)
}
```
* 具体例として、下記ではinterface{}型はerrorとして利用することはできない。
```go
package main

func main() {
	f0(getErr())// こちらはOK: 実体がnilでもerror型として渡すことができる
	f1(getErr())// こちらはpanic
}

func getErr() error {
	// nilをerrorとして返すことができる。
	return nil
}

func f0(a error) error {
	return a// error型として受け取っているので、error型として返すことができる
}

func f1(a any) error {
	// return a// error型ではないのでコンパイルエラー
	return a.(error)// panic: any型として受け取ると、もうerror型にすることはできない。
}
```



# インターフェース型のnilについて
* interface型のnilは大きく分けると2つある点に注意
	1. inteface型のゼロ値: 定義にあるようにtypeとdataの両方がnilである場合
	2. typeに値が入っていて、dataがnilの場合
		* これは typed nilと呼ばれる。
		* 公式の用語では無いと思われるが、便宜上メモではtyped nilと表記する。
* 注意したい点として、上記の1.と2.は比較した際に一致しないことである。
* https://go.dev/doc/faq#nil_error
* https://go-tour-jp.appspot.com/methods/12
* サンプルコード1
	```go
	package main
	import (
		"fmt"
		"reflect"
	)
	type myError struct {
		message string
	}
	func (e *myError) Error() string { // myErrorはerror interfaceを実装している。
		return e.message
	}
	func main() {
		var nilErr error
		fmt.Println("is nil:", nilErr == nil)          // true
		fmt.Println("Type:", reflect.TypeOf(nilErr))   // <nil>
		fmt.Println("Value:", reflect.ValueOf(nilErr)) // <invalid reflect.Value>

		var p *myError
		fmt.Println("is nil:", p == nil) //true

		var notNilErr error = p
		fmt.Println("is nil:", notNilErr == nil)          // これは、false
		fmt.Println("Type:", reflect.TypeOf(notNilErr))   // *main.myError
		fmt.Println("Value:", reflect.ValueOf(notNilErr)) // myError: <nil>

	}
	```
* サンプルコード2
	```go
	package main

	type MyInt int32
	type MyStruct struct{}
	type i interface{}
	type MyStruct2 struct{ i }

	func hoge() *MyStruct {
		var a *MyStruct
		return a
	}
	func hoge2() i {
		var a *MyStruct2
		return a
	}
	func fuga(a interface{}) {
		println(a == nil)
	}
	func main() {
		var a *int32
		var b *MyInt
		var c *MyStruct
		var d *MyStruct2
		println(a == nil)                     // true: 通常は比較してtrueとなる
		println(b == nil)                     // true: 通常は比較してtrueとなる
		println(c == nil)                     // true: 通常は比較してtrueとなる
		println(d == nil)                     // true: 通常は比較してtrueとなる
		println(hoge() == nil)                // true: interface型にはなっていないので比較してtrueとなる
		println(hoge2() == nil)               // false: inteface型であり、型が一致しないとfalseとなる。
		println(hoge2() == (*MyStruct2)(nil)) // true: inteface型であり、型が一致するのtrueとなる。
		fuga(a)                               // false: inteface型として渡され、型が一致しないためfalseとなる。
		fuga(nil)                             // true: inteface型として渡され、型がないためtrueとなる。
		fuga((*int32)(nil))                   // false: inteface型として渡され、 型が一致しないためfalseとなる。
		fuga((error)(nil))                    // true: errorはinterfaceだからtrueになる？
	}
	```
* 具体例として、errorのインターフェースとして返す際に注意が必要。
* errorのインターフェースとなった時点で、nilの比較は型情報まで含まれてしまうため。
* nilを返す際は型を持たないnilを返すようにすることが、通例となる。
```go
package main

type MyError struct{}

func (e *MyError) Error() string { return "error!" }
func hoge() error {
	var myErr *MyError // この時点ではmyErr == nil
	//　処理中にエラーがあればmyErrに代入することを想定。
	return myErr
}

func fuge() error {
	return (*MyError)(nil)
}

func main() {
	println(hoge() == nil)             // falseになってしまう！
	println(hoge() == (*MyError)(nil)) // true

	println(fuge() == nil)             // falseになってしまう！
	println(fuge() == (*MyError)(nil)) // true
}
```
* 下記のようにnilを返すと良い。
```go
if エラーを検出 {
		return &MyError{}
	}
// 正常終了
return nil
```
* https://tutuz-tech.hatenablog.com/entry/2019/10/22/162231
* https://zenn.dev/nobonobo/articles/f554041aea1955
* https://bleis-tift.hatenablog.com/entry/go-the-bad-parts
* https://zenn.dev/hsaki/articles/go-convert-json-struct
* https://zenn.dev/nobonobo/articles/19c84c231aff46
## typed nilはメソッドを実行してもpanicとならない
* 例えばnilのポインターの先を取得しようとすると、エラーとなる。
* 一方で、interfaceは型情報を持っているnilであればpanicとはならない。
* https://go.dev/tour/methods/12
```go
package main

type i interface {
	M()
}

type t string

func (tt *t) M() {
}

func main() {
	var ii i
	// ii.M()//  これは型情報の無いnilのためpanicとなる。
	var tt *t
	ii = tt
	ii.M()// これは型情報が入っているnilのためpanicとならない。
}
```



# 型アサーション（Type Assertions）
* https://go-tour-jp.appspot.com/methods/15
* interface{}は具象値の型情報が含まれる。
* したがって、Interface()で受け取った値に型アサーションすることで、値を復元できる。
* 以下のようにすると、変数 s は文字列型になる。（型が間違っていると、パニックになる）
```go
i := interface{}("hello")
s := i.(string)
```
```go
var x float64 = 3.4
v := reflect.ValueOf(x) 
y := v.Interface().(float64) // y will have type float64.
fmt.Println(y)
```
* パニックにならないように、下記で確認できる。
```go
 i := interface{}("hello")
n, ok := i.(int)
fmt.Println(n, ok) // 0  false
```
## 参考
* ConnPoolがinterfaceとしてPing()を持っているかアサーション。
```go
if pinger, ok := db.ConnPool.(interface{ Ping() error }); ok {
    err = pinger.Ping()
}
```
* driveriがinterfaceのdriver.DriverContextを満たしているかアサーション
```go
if driverCtx, ok := driveri.(driver.DriverContext); ok {
    ...
}
```


# Type switches
* interface{}型に対しては、型switchができる。
```go
i := interface{}("hello")
switch i.(type) {
	case string:
		fmt.Println("string")// string
	default:
		panic("")
}
```
* https://go-tour-jp.appspot.com/methods/16
* 例）gormのselect関数の実装
```go
func (db *DB) Select(query interface{}, args ...interface{}) (tx *DB) {
	...
    switch v := query.(type) {
	case []string:
		tx.Statement.Selects = v // この時点で vは []string型として扱える。
		for _, arg := range args {
			switch arg := arg.(type) {
			case string:
				...
			case []string:
				...
			default:
				...
			}
		}
    ...
	case string:
	...
}
```
## caseの注意
```go
switch vv := val.(type) {
	case bool:
		return GetZeroVal(vv)// これだと、vvは bool値になるが、
	case int, string:
		return GetZeroVal(vv)// これだと、vvは any型になる。（したがって意図した結果にならない。）
}
func GetZeroVal[R any](a R) R {
	return *new(R)
}
```

# よく使われる標準パッケージのインターフェース
* fmt.Stringer
	* fmtパッケージに定義されている
	* fmt.printlnだと、Stringerインターフェースのstringメソッドが呼ばれている。
	* https://go-tour-jp.appspot.com/methods/17
* fmt.Error
	* 関数はよくerror値を返す。error値がnilでない場合はエラーである。
	* https://go-tour-jp.appspot.com/methods/19
* io.Reader
	* データストリームを読むことを表現する
	* 実装例として、例えば、r := strings.NewReader("Hello, Reader!") して、 for 文で　n, err := r.Read(b) 
	* このReadは 渡したバイトスライス（b）にデータを入れて、入れたバイトのサイズとエラーの値を返す。
* image.Image
	* https://go-tour-jp.appspot.com/methods/24


# interfaceを実装した変数のポインター、の表現
* まず、ポインタじゃないケースは、例えば下記のように当然いけるのだが、
```go
type i interface{ f() }
type s struct{}

func (ss s) f() {}
func o(arg i)   {}

func main() {
	o(s{})
    // o(&s{}) //なんかこれでも行けた。
}
```
* これは駄目。o(arg *i) の *i は 「iを実装したクラスのポインター」ではなく、「iへのポインター」になるようだ。
```go
type i interface{ f() }
type s struct{}

func (ss s) f() {}
func o(arg *i)   {}
// func o[T i](arg *T) {} // これだと行ける。

func main() {
	// o(&s{}) //エラー
	o(new(i)) //　*iを渡すと通る。
}
```



# interfaceの埋め込み
* interface に interfaceへ埋め込むこともできる
	* https://go.dev/ref/spec#Embedded_interfaces
	```go
	package main

	type MyInteface interface {
		Test()
	}

	type MyInteface2 interface {
		MyInteface
	}

	type Usecase string

	func (u Usecase) Test() {}

	func main() {
		F1(Usecase(""))
	}

	func F1(a MyInteface2) {
		a.Test()
	}
	```
* interfaceを構造体へ埋め込むことができる
  * https://stackoverflow.com/questions/26027350/go-interface-fields
	```go
	package main

	type I interface {
		Test()
		Test2()
	}

	type S struct {
		I
	}

	func (s S) Test2() {}

	func main() {
		F1(S{})
	}

	func F1(a I) {
		//a.Test()//実装していない場合は実行時エラーとなる
		a.Test2()
	}
	```


# General interfaces
* TODO: この部分についてドキュメントを読んだけど、十分に理解できていない。
* https://go.dev/ref/spec#General_interfaces
* 最も一般的なinterfaceの形として、インターフェース要素は、任意の型項T、または基礎となる型Tを指定する ~Tの形の項、または項t1|t2|...|tnの和
* メソッドの指定とともに、これらの要素によって、インターフェースの型集合を以下のように正確に定義することができる
	* 空のインターフェイスの型集合は、すべての非インターフェイス型の集合である。
	* 空でないインターフェイスの型集合は、そのインターフェイス要素の型集合の交点である。
	* メソッド指定の型集合は、そのメソッドをメソッド集合に含むすべての非インタフェース型の集合である。
	* 非インタフェース型の項の型集合は、その型だけからなる集合である。
	* ~Tという形式の項の型集合は、基礎となる型がTであるすべての型の集合である。
	* 項t1|t2|...|tnの和の型集合は、その項の型集合の和である。
```go
// An interface representing only the type int.
interface {
	int
}

// An interface representing all types with underlying type int.
interface {
	~int
}

// An interface representing all types with underlying type int that implement the String method.
interface {
	~int
	String() string
}

// An interface representing an empty type set: there is no type that is both an int and a string.
interface {
	int
	string
}
```
* ~Tという形の項では、Tの基礎となる型はそれ自身でなければならず、Tはインターフェイスであってはならない。
	```go
	type MyInt int

	interface {
		~[]byte  // the underlying type of []byte is itself
		~MyInt   // illegal: the underlying type of MyInt is not MyInt
		~error   // illegal: error is an interface
	}
	```
* union要素は型集合のunionを表す
	```go
	// The Float interface represents all floating-point types
	// (including any named types whose underlying types are
	// either float32 or float64).
	type Float interface {
		~float32 | ~float64
	}
	```
* Basic interfacesではないinterface型は、値や変数の型や、他の非インタフェース型の構成要素になることはできない。
	* Basic interfacesとは、下記のようなもの。
		* https://go.dev/ref/spec#Basic_interfaces
		* https://stackoverflow.com/questions/71073365/interface-contains-type-constraints-cannot-use-interface-in-conversion

		```go
		interface {
			Read([]byte) (int, error)
			Write([]byte) (int, error)
			Close() error
		}
		```
	```go
	var x Float                     // illegal: Float is not a basic interface
	var x interface{} = Float(nil)  // illegal
	type Floatish struct {
		f Float                 // illegal
	}
	```


# generics
* 以下は、ポインターであり、かつ GetIDというメソッドを持つ構造体を表現
  * genericsを使うことで、「任意の型」を表現できている。
```go
type HasID[T any] interface {
	GetID() string
	*T
}
```
* 以下のPointerは任意のポインターを表現
```go
type Pointer[T any] interface {
	*T
}
```
* ただ、上記だとHasIDを使う側で毎回余計なジェネリクスを一つ増やさないといけないので
    * 「任意の型のポインタであること」を表現できるとありがたい。
* https://stackoverflow.com/questions/71547015/how-to-define-type-constraint-on-pointer-receiver-ingolang
* https://go.dev/play/p/myw6FAosPjK


