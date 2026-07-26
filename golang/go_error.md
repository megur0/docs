---
title: "エラーハンドリング - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > エラーハンドリング




# 間違ったエラーハンドリング
```go
if strings.Contains(err.Error(), "cannot unmarshal") {
    logger.Warn(c.RequestID, "Invalid field format:%s", err)
    return ErrRequestFieldFormat
}
```

# panic, recover

## panic
* programを non-zeroのコードでexitさせる。
* deferは実行され、呼び出し元のdeferも実行される。
    * os.Exitは呼ばれないため、Fatalログ + os.Exitより、panicの方が良い気がする。
* スタックトレースも出力される。
```go
func f1() {
	defer print("f1 defer\n") // deferは呼ばれる。
	panic("f1 panic!!!!")
	print("not execute!!!") // 実行されない
}
func f2() {
	defer print("f2 defer\n") //呼び出し元のdeferも呼ばれる。
	f1()
}
func main() {
	f2()
	print("not execute!!!") // 実行されない
}
// f1 defer
// f2 defer
// panic: f1 panic!!!!
```


## recover
* panicが起きるような自体においてはプログラムは落ちたほうが良い状況なので、基本recoverはしない。
* ただ、net/httpでは、リクエストハンドラがpanicしたのをrecoverして単一のHTTPリクエストがプロセス全体を終了させないようにしているので、こういう使い所はある。
* https://qiita.com/ruiu/items/ff98ded599d97cf6646e#2-2
* https://go.dev/doc/effective_go#errors
* https://zenn.dev/nobonobo/articles/e0af4e8afc6c38b42ae1
### recover() は関数内部で実行されることで初めてpanicを受け取ってくれる
* これだとpanicは回収されない
```go
func main() {
	recover()
	panic("パニックが回収されませんでした")
}
```
* 回収される
```go
func main() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("パニックが回収されました")
		}
	}()
	panic("パニックが回収されませんでした")
}
```
### panic時に戻り値はデフォルト値になる。
```go
func main() {
	s := f()
	fmt.Printf("%#v\n", s)// <nil>になる。
}
func f() error {
	var r interface{}
	defer func() {
		r = recover()
		if r != nil {
			println(r.(string))
		}
	}()
	if true {// 分岐を挟んでいるのは、go vet による nerver reached codeの回避のため。
		panic("fff")
	}
	return errors.New("aaa") // ここは実行されない。
}
```




# error, errors（Wrap, Is, As）


## Goのエラー周りの参考
* https://zenn.dev/link/comments/1dd0f100389e4e

## interface
```go
// /usr/local/go/src/builtin/builtin.go
type error interface {
	Error() string
}
```

## errors.New
```go
// /usr/local/go/src/errors/errors.go
func New(text string) error {
	return &errorString{text}
}

type errorString struct {
	s string
}
```

## Goのエラーの返しには暗黙のルールがある
* https://zenn.dev/nobonobo/articles/19c84c231aff46
* エラーがnilではない時、処理結果が保有するリソースは開放する必要がない
* エラーがnilの時、処理結果が保有するリソースは不要になったらリソース解放処理の呼び出しが必要
* エラーがnilの時、処理結果がポインタ型である場合このポインタはnilではないことを期待していい。
* nilでないエラーを返しつつ解放が必要なリソースも返す様な処理関数やメソッドは存在するのか？
    * これは「存在しないことが期待されている」

## エラーのラップ
* interface{ Unwrap() error }を実装しておくことと、errors.Isやerrors.AsがUnwrapの戻り値のerrorを再帰的にチェックしてくれる。
	* この仕組みによってエラーのラップが実現できる。
### ラップ方法
* fmt.Errorf()を使う方法
	* fmt.Errorf()で”%w”フォーマットを適用すると、wrapError構造体ポインタが返される。
	* wrapErrorは下記のように定義されている。
		```go
		type wrapError struct {
			msg string
			err error
		}
		func (e *wrapError) Unwrap() error {
			return e.err
		}
		```
	* 例
		```go
		func main() {
			fmt.Println(fmt.Errorf("wrapping: %w", testErr()))
		}
		func testErr() error {
			return errors.New("test error")
		}
		```
* 自前で定義する。
	* 例えば、下記のErrRequestJsonSomethingInvalidは json.Unmarshal を実行した時に戻ってきたerrorをラップ(ErrRequestJsonSomethingInvalid.Errにセット)して、
	```go
	type ErrRequestJsonSomethingInvalid struct {
		Json string
		Err  error
	}
	func (e *ErrRequestJsonSomethingInvalid) Error() string {
		return fmt.Sprintf("json is something invalid: %s", string(e.Json))
	}
	func (e *ErrRequestJsonSomethingInvalid) Unwrap() error {
		return e.Err
	}

	```
* deferを使う例
	* https://kaminashi-developer.hatenablog.jp/entry/2022/12/15/093000


## errors.Is, errors.As
### errors.Is(err, target error) bool
* errとtargetの単純比較、もしくはerr.Is(error) boolが実装されていれば、その結果を返す。
* err.Unwrap() errorが実装されている場合は、再帰的にUnwrapの結果をerrに代入して上と同様の比較を行う。
### errors.As(err error, target any) bool
* 以下のいずれかでpanicとなる。(つまり、targetはerror型のポインタ or インターフェースへのポインタ である必要がある。)
	* targetがnil
	* targetがポインターではない。
	* targetがポインターだが、先の値がnil
	* targetポインタの先が、インターフェースではない、かつ、errorメソッドを実装していない 
* 以下のいずれかでtrue
	* errの値がtargetポインターの先の値に代入可能である
		* この場合はtargetが指す先にerrが代入される。
	* errがAsメソッドを定義しているかつ、targetを引数に取ったAsメソッドの結果がtrue
* err.Unwrap() errorが実装されている場合は、再帰的にUnwrapの結果をerrに代入して上と同様の比較を行う。
### なんとなくの使い方。
* Is -> 一致するかどうか。Isを実装することで一致条件をカスタマイズできる。
* As
	* 同系統の複数のエラーをひとつの構造でまとめて扱いたいとき。
		* 代入可能であればerrors.Asはtrueとなるため、あるエラーをtypeで宣言してerrorを実装しておき、それを複数のエラーで利用する。
* いずれも、Unwrapを実装しておくことでラップした中身まで再帰的に比較できる。
* https://zenn.dev/jundayo/articles/b5c6fd7d3e50f0



```go
import (
	"encoding/json"
	"errors"
	"strings"
	"testing"

	"github.com/google/uuid"
	"github.com/stretchr/testify/assert"
)

func Ptr[T any](a T) *T {
	return &a
}

func TestError(t *testing.T) {
	t.Run("assert_error", func(t *testing.T) {
		//errors.IsはDeep equalではない。したがってこれはfalseになる
		assert.Equal(t, errors.Is(errors.New(""), errors.New("")), false)
		a := errors.New("")
		assert.Equal(t, errors.Is(a, a), true) // これはtrue
		// AssertFalse(t, errors.As(errors.New(""), Ptr(errors.New("")))) これだとanalyzerのerrorになる。（第二引数がerror型は駄目らしい）

		// Unmarshalで発生するエラーの判定の例
		assert.Equal(t, errors.As(json.Unmarshal([]byte("{"), &struct{}{}), Ptr(&json.SyntaxError{})), true)// json.SyntaxErrorはポインタレシーバーのため、&json.SyntaxError{}を渡す必要があり、errors.Asの第二引数targetはerror型へのポインタである必要があるため、Ptr(&json.SyntaxError{})を渡す。
		assert.Equal(t, errors.As(json.Unmarshal([]byte(`{"id":"a"}`), &struct {
			Id int `json:"id"`
		}{}), Ptr(&json.UnmarshalTypeError{})), true)
		assert.Equal(t, errors.As(json.Unmarshal([]byte(`{"id":"a"}`), (*int)(nil)), Ptr(&json.InvalidUnmarshalError{})), true)

		// uuidの実装はエラーの定義がないため、errros.Isで比較できないため文字列で比較する必要がある。
		strings.Contains(json.Unmarshal([]byte(`{"id":"a"}`), &struct {
			Id uuid.UUID `json:"id"`
		}{}).Error(), "invalid UUID length")
		assert.Equal(t, json.Unmarshal([]byte(`{"id":"g0000000-0000-0000-0000-000000000000"}`), &struct {
			Id uuid.UUID `json:"id"`
		}{}).Error(), "invalid UUID format")
	})
}
```
