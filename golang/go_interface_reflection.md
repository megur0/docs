---
title: "リフレクション - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > リフレクション





# リフレクション
* https://go.dev/blog/laws-of-reflection
* リフレクションは、interfaceの変数内に格納されている型と値のペアを検査するためのメカニズム


# reflect.ValueOf
* reflect.Valueを返す。
* KindやTypeメソッドが使える。
	* Kind()は Kind型を返す。
		* type Kind uint
		* reflect.Float64 とかと比較ができる。
	* Type()はValue.typ（rtype型）Typeとして返す。
		* reflect.Type
			* type Type interface
				* Kind() Kind
				* Elem() Type
				* Field(i int) StructField
				* ...
```go
var x float64 = 3.4
v := reflect.ValueOf(x) // reflect.Valueを取得
fmt.Println("type:", v.Type())
fmt.Println("kind is float64:", v.Kind() == reflect.Float64)
//type: float64
//kind is float64: true
```
* Float()やString()などで基底の値を取得できる。（型が違うとpanicになることがある）
```go
v := reflect.ValueOf("test")
	fmt.Println(v)          // test
	fmt.Println(v.String()) // test
	//fmt.Println(v.Int())    // panic
	fmt.Println(v.Type())   // string
```
*  IsNil()
参照型の参照先の値がnilかどうか確認
```go
type myError struct {
	message string
}
func (e *myError) Error() string { // myErrorはerror interfaceを実装している。
	return e.message
}

func main() {
	// 参照型以外に使うとエラーになる。
	//a := 2
	//fmt.Println(reflect.ValueOf(a).IsNil()) // panic

	b := []string{"aaa"}
	fmt.Println(reflect.TypeOf(b))          // []string
	fmt.Println(reflect.ValueOf(b))         // [aaa]
	fmt.Println(reflect.ValueOf(b).IsNil()) // false

	m := map[string]string{}
	fmt.Println(reflect.TypeOf(m))          // map[string]string
	fmt.Println(reflect.ValueOf(m))         // map[]
	fmt.Println(reflect.ValueOf(m).IsNil()) // false

	var p *myError        // myErrorはerrorを実装。
	fmt.Println(p == nil) //true
	var notNilErr error = p
	fmt.Println(notNilErr == nil)                   // false
	fmt.Println(reflect.TypeOf(notNilErr))          // *main.myError
	fmt.Println(reflect.ValueOf(notNilErr))         // <nil>
	fmt.Println(reflect.ValueOf(notNilErr).IsNil()) // true
	fmt.Println(reflect.ValueOf(notNilErr).Elem())  // <invalid reflect.Value>
	//fmt.Println(reflect.ValueOf(notNilErr).Elem().IsNil()) // panic
}
```
* reflect.Valueの構造
```go
type Value struct {
	typ *rtype
	ptr unsafe.Pointer
	flag
}
```

# reflect.TypeOf
* TypeOfは値をreflect.Typeを返す
    * TypeOfの引数の型はinterface{}になっている。
```go
var x float64 = 3.4
fmt.Println("type:", reflect.TypeOf(x))
```


# Interface()
* interface{}型として取得する。（これを、unpackと言うらしい。）
	* InterfaceメソッドはValueOf関数の逆と考えることができる。
* そのため、reflect.ValueOfで取得した値はいつでも復元できる。
```go
var x float64 = 3.4
v := reflect.ValueOf(x) 
y := v.Interface().(float64) // y will have type float64.
fmt.Println(y)
```
* fmt.Printはもともと引数の型がinterface{}であり、内部でそれを復元しているため、interfaceのまま渡せる。
```go
var x float64 = 3.4
v := reflect.ValueOf(x)
fmt.Println(v.Interface())
```
* なお、現在のfmtは自動的にreflectの値をunpackするため、reflect.valueのまま渡しても動作する。
```go
fmt.Println(v)
```

# リフレクトから値を得る手段
* String()などのメソッドで取得する方法
```go
fmt.Println(reflect.ValueOf("test").String()) // test
```
* interfaceに変換して型アサーションする。
```go
fmt.Println(reflect.ValueOf(time.Now()).Interface().(time.Time))
```
* 型が不明な場合はswitchで。
```go
switch v := rv.Interface().(type) {
	case string:
		if v == "" {
			return false
		}
	case int:
		if v == 0 {
			return false
		}
	case time.Time:
		if v.IsZero() {
			return false
		}
	case *string, *int, *time.Time:
		if v == nil {
			return false
		}
	default:
		panic(fmt.Sprintf("unexpected type: %#v", rv))
	}
```


# Settable
* reflect.Valueは、interface{}としてコピーした値であり、
* Set***メソッドで値を設定することはできない。
```go
var x float64 = 3.4
v := reflect.ValueOf(x)
fmt.Println("settability of v:", v.CanSet()) // false
v.SetFloat(7.1) // Error: will panic.
```
* ポインタをValueOfに渡し、かつ、Elemで受け取ったreflect.Valueを使う必要がある。
```go
var x float64 = 3.4
p := reflect.ValueOf(&x) // Note: take the address of x.
fmt.Println("type of p:", p.Type())
fmt.Println("settability of p:", p.CanSet())               // false
fmt.Println("settability of p.Elem():", p.Elem().CanSet()) // true
p.Elem().SetFloat(7.1)
fmt.Println(p.Interface())        // pointer value
fmt.Println(p.Elem().Interface()) // 7.1
fmt.Println(x)                    // 7.1
```

# reflect.Value.Elem()、reflect.Type.Elem()
```go
k := 3
vk := reflect.ValueOf(k)
fmt.Println(vk.Type()) // int
// Elem()はreflect.Valueの型が参照型じゃない場合はpanicになる。
// fmt.Println(vk.Type().Elem()) // panic
// fmt.Println(vk.Elem()) // panic

// slice型でもエラーになるようだ。
b := []string{"aaa"}
fmt.Println(reflect.TypeOf(b)) // []string
// fmt.Println(reflect.ValueOf(b).Elem()) // panic

var j *int
vj := reflect.ValueOf(j)
// nilであるポインターのValueでも、Type、Type().Elem()はエラーにならない。
fmt.Println(vj.Type()) // *int
fmt.Println(vj.Type().Elem()) // int
// nilであるポインターのValueでも、Elem()はエラーにならないがinvaldが返ってくる。
fmt.Println(vj.Elem())        // <invalid reflect.Value>
// invalidな値に対してType()を実行するとpanicになる。
// fmt.Println(vj.Elem().Type()) // panic

j = &k
vj = reflect.ValueOf(j)
fmt.Println(vj.Type().Elem()) // int
fmt.Println(vj.Elem())        // 3

```
* https://future-architect.github.io/articles/20220921a/


# 構造体
```go
type T struct {
    A int
    B string
}
t := T{23, "skidoo"}
s := reflect.ValueOf(&t).Elem()// Elem()で取得したreflect.ValueじゃないとSetができない。
for i := 0; i < s.NumField(); i++ {
    f := s.Field(i) // reflect.Valueを取得
    sf := s.Type().Field(i) // reflect.StructFieldを取得。（※本当はforの外部でs.Type()は取得しておいたほうが効率的）
    fmt.Printf("%d: %s %s = %v\n", i, sf.Name, f.Type(), f.Interface())
}

s.Field(0).SetInt(77)                // &tを渡している、かつElem()で取得しているため、セットできる。
s.Field(1).SetString("Sunset Strip") // &tを渡している、かつElem()で取得しているため、セットできる。
fmt.Println("t is now", t)

var a = 2
b := reflect.ValueOf(a)
s.Field(0).Set(b) // Setでreflect.Valueをセットできる。
fmt.Println("t is now", t)
```
## FieldByName
```go
s := struct{ Name string }{}
vps := reflect.ValueOf(&s).Elem()
vf := vps.FieldByName("Name")
fmt.Println(vf.Type(), vf.CanSet(), vf.CanAddr()) // string　true true
```
* 例えばgo-cmpのパッケージで、比較のオプションで特定のフィールドを無視することができるが、
	* この内部実装では、FieldByNameが使われている。
	* cmpopts.IgnoreFields(Compare{}, "CreatedAt", "UpdatedAt")


# New
```go
v := reflect.New(reflect.TypeOf(0))
fmt.Println(v)        // 0x...
fmt.Println(v.Type()) // *int   Newで返されるreflect.Valueの基底はポインターになる。
fmt.Println(v.Elem()) // 0

a := reflect.New(reflect.TypeOf(sql.NullString{}))
fmt.Printf("%v", a) // &{ false}
```
* 参考
	* https://ja.stackoverflow.com/questions/66039/go-と-database-sql-で構造体がない場合でも値を取得したい


# Indirect
* reflect.Valueの値がポインターなら、其の参照先を取得し、ポインターじゃないならそのまま。
* structなどで、ポインターかそうではない場合どちらの可能性もある場合に、型をチェックする必要がないので便利。
* https://qiita.com/chimatter/items/b0879401d6666589ab71


# ElemやIndirectで取得したreflect.ValueのTypeが、 reflect.ValueOfとふるまいが違う件
```go
func main() {
	var a any // 型が any。
	a = 4 // 中身はint
	fmt.Println(reflect.ValueOf(a).Type().Kind())         // これは中身の型のintをみる。
	fmt.Println(reflect.ValueOf(&a).Type().Kind())        // ptr
	fmt.Println(reflect.ValueOf(&a).Elem().Type().Kind()) // これは宣言時の型のinterfaceになる。
	fmt.Println(reflect.Indirect(reflect.ValueOf(&a)).Type().Kind()) // これもinterface
	
	
	fmt.Println(reflect.ValueOf(reflect.ValueOf(&a).Elem()).Type().Kind())// これだとstruct　になってしまう（なぜ？？）
	fmt.Println(reflect.ValueOf(reflect.Indirect(reflect.ValueOf(&a))).Type().Kind()) // これもstruct

}
```


# その他参考
https://kaminashi-developer.hatenablog.jp/entry/2022/11/08/095537
https://qiita.com/masakurapa/items/e1e49f9d6c864d97458f
https://maku77.github.io/p/hxhzfbs/



# サンプルコード
```go
import (
	"reflect"
	"testing"
	"time"

	"github.com/stretchr/testify/assert"
)

func Ptr[T any](a T) *T {
	return &a
}

func TestReflect(t *testing.T) {
	t.Run("assert_reflect", func(t *testing.T) {
		assert.Exactly(t, reflect.TypeOf(Ptr(0)).Kind(), reflect.Ptr)
		assert.Exactly(t, reflect.TypeOf((*int)(nil)).Kind(), reflect.Ptr)
		assert.Exactly(t, reflect.TypeOf(Ptr(0)).Elem().Kind(), reflect.Int)
		assert.Exactly(t, reflect.TypeOf((*int)(nil)).Elem().Kind(), reflect.Int)
		assert.Exactly(t, reflect.ValueOf(nil).Kind(), reflect.Invalid)
		assert.Exactly(t, reflect.ValueOf((*int)(nil)).Kind(), reflect.Pointer)
		assert.Exactly(t, reflect.ValueOf((*int)(nil)).Type().Kind(), reflect.Pointer)
		assert.Exactly(t, reflect.ValueOf((*time.Time)(nil)).Type().Kind(), reflect.Pointer)
		assert.Exactly(t, reflect.ValueOf((*int)(nil)).Elem().Kind(), reflect.Invalid)
		assert.Exactly(t, reflect.ValueOf((*int)(nil)).Type().Elem().Kind(), reflect.Int)

		assert.Exactly(t, reflect.New(reflect.TypeOf(0)).Elem().Int(), (int64)(0))
		assert.Exactly(t, *reflect.New(reflect.TypeOf((*time.Time)(nil)).Elem()).Interface().(*time.Time), time.Time{}) // nilからでも生成することができる。
		assert.Exactly(t, *reflect.New(reflect.ValueOf((*time.Time)(nil)).Type().Elem()).Interface().(*time.Time), time.Time{})

		assert.Exactly(t, reflect.ValueOf(nil).CanAddr(), false)
		assert.Exactly(t, reflect.ValueOf(time.Time{}).CanAddr(), false)
		assert.Exactly(t, reflect.ValueOf(&time.Time{}).CanAddr(), false)
		assert.Exactly(t, reflect.ValueOf(&time.Time{}).Elem().CanAddr(), true)
		assert.Exactly(t, reflect.ValueOf((*int)(nil)).Elem().CanAddr(), false)
		assert.Exactly(t, reflect.ValueOf((*time.Time)(nil)).Elem().CanAddr(), false)
		assert.Exactly(t, reflect.ValueOf(&struct{ t time.Time }{}).Elem().CanAddr(), true)
		assert.Exactly(t, reflect.ValueOf(&struct{ t time.Time }{}).Elem().Field(0).CanAddr(), true)
		assert.Exactly(t, reflect.ValueOf(&struct{ t *time.Time }{}).Elem().Field(0).CanAddr(), true)
		assert.NotEqual(t, reflect.ValueOf(&struct{ t *time.Time }{}).Elem().Field(0).Addr(), nil)
	})
}
```