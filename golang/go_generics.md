---
title: "ジェネリクス - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > ジェネリクス



### ドキュメント
* https://go.dev/ref/spec#Type_parameter_declarations
* チュートリアル
  * https://go.dev/doc/tutorial/generics
* プロポーザル
  * https://go.googlesource.com/proposal/+/refs/heads/master/design/43651-type-parameters.md

## 参考
* https://zenn.dev/nobishii/articles/type_param_intro
* https://recursionist.io/learn/languages/go/oop/generics


## ジェネリクス(generics)()
* 1.18以降、利用可能
* 関数、型において型パラメータを利用することができる
* 型パラメータはinterface型によって制約を持たせる



## nilを指定すると ジェネリクスを推論できないケースの対応。
* 「can not infer RES」とエラーが出る。ジェネリクスを推論できないからだ。
```go
...
  return handle(c, &request.DeclineFriendRequest{}, (nil, http.StatusOK, func(req *request.DeclineFriendRequest) (any, error) {...})
...

func handle[REQ, DATA any, RES ResponseHasSet[DATA]](c echo.Context, request *REQ, res RES, successStatusCode int, logic func(*REQ) (DATA, error)) error {
	...
}
```  
* nilをキャストする方法。
```go
...
  return handle(c, &request.DeclineFriendRequest{}, (*response.EmptyData)(nil), http.StatusOK, func(req *request.DeclineFriendRequest) (any, error) {...})
```
* 明示的に指定する方法。欠点はこのケースでは長くなってしまうこと。inferできるものを省略する記法があれば良いのだが、分からなかった。
```go
return handle[request.DeclineFriendRequest, any, *response.EmptyData](c, &request.DeclineFriendRequest{}, nil, http.StatusOK, func(req *request.DeclineFriendRequest) (any, error) {...})
```






## comparable
* https://zenn.dev/nobishii/articles/3-means-of-go-comparable
* Go1.18のジェネリクス導入により、事前宣言された型制約comparableが導入された。
* 型Xがcomparableを実装するとは、下記のどちらか。
  * Tがインターフェイス型ではなく、かつTが==と!をサポートしている
  * Tがインターフェイス型であり、Tの型集合の各型が同等のものを実装している。
```go
func main() {
	var x int
	var y any
	f(x) // ok
	// f(y) // compile error
	_ = y
}
func f[T comparable](x T) {}
```

### any型 は comparable ではない。
* T any ってした場合は Tの変数を比較できないので注意。比較したいならcomparableを使う。
* https://stackoverflow.com/questions/68053957/go-with-generics-type-parameter-t-is-not-comparable-with
