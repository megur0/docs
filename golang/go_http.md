---
title: "http - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > http





# net/httpによるサーバー参考
https://goodbyegangster.hatenablog.com/entry/2022/10/07/122943
content-typeとかのチェックもある。
https://andmorefine.gitbook.io/learn-go-with-tests/build-an-application/json


# net/http  
* echoやginやgorillaなどはnet/httpのラッパー的な感じ。
* 参考
  * https://journal.lampetty.net/entry/understanding-http-handler-in-go
  * https://tech.yappli.io/entry/2022/05/16/Goのnet/httpパッケージを理解する
  * https://future-architect.github.io/articles/20210714a/

# 各種例
* テストコードを参照。

# httpクライアント
* 以下の２つの方法がある。
  * net/httpパッケージに定義されている関数を使用する方法
    * http.Get関数とか
  * net/httpパッケージに定義されているClient構造体に実装されたメソッドを使用する方法
    * Client構造体のGetメソッドを使用する、とか
* Client構造体
  * HTTPトランスポート
  * リダイレクトを処理するためのポリシー
  * クッキージャー（Jarは送信リクエストにクッキーを挿入するために使用され、レスポンスのクッキー値で更新される）
  * リクエストの制限時間（タイムアウト）（0の場合は制限しないことを指す）
* https://leben.mobi/go/http-client-get/practice/web/

# text/template　で テンプレートをparse
* https://qiita.com/taizo/items/bf1ec35a65ad5f608d45
```go
// 実行
// go run server.go
// open http://localhost:8080
import (
  "net/http"
  "text/template"
)
func main() {
  http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    tmpl, err := template.New("new").Parse("{{.Title}} {{.Count}} count")
    if err != nil {
      panic(err)
    }
    err = tmpl.Execute(w, struct { 
      Title string
      Count int
    }{"Hello World.", 1}) 
    if err != nil {
      panic(err)
    }
  })
  http.ListenAndServe(":8080", nil)
}
```
* ファイルを使う方法
```go
func main() {
  http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    page := struct {
      Title string
      Count int
    }{"Hello World.", 1}
    tmpl, err := template.ParseFiles("layout.html")
    if err != nil {
      panic(err)
    }
    if tmpl.Execute(w, page); err != nil {
      panic(err)
    }
  }) // hello

  http.ListenAndServe(":8080", nil)
}
```
```html
//layout.html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>{{.Title}}</title>
  </head>
  <body>
    {{.Title}} {{.Count}} count
  </body>
</html>
```



# ミドルウェア
```go
func middlewareOne(next http.Handler) http.Handler {
  return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    // Our middleware logic goes here...
    next.ServeHTTP(w, r)
  })
}
http.Handle("/", middlewareOne(middlewareTwo(finalHandler)))
```
* https://tutuz-tech.hatenablog.com/entry/2020/03/23/220326



## context
* http.Requestにはcontext.Contextが入っている。
* これはキャンセル情報なども伝搬してくれる。（httpコネクションの切断 等）
* キャンセルの伝搬を処理する？
	* 個人的に複雑性が増すためキャンセルハンドリングはしていない。
* 値のセット
	* context.WithValueで新たに生成したctxを持ったhttp.Requestを新たに生成する必要がある。
	* http.Request.WithContext を使う。
	```go
	ctx := context.WithValue(r.Context(), "user-id", user)
	r = r.WithContext(ctx)
	```
* https://medium.com/@agatan/http%E3%82%B5%E3%83%BC%E3%83%90%E3%81%A8context-context-7211433d11e6



# Graceful shutdown
* https://cloud.google.com/blog/ja/products/application-development/graceful-shutdowns-cloud-run-deep-dive
* ソースコードにいろいろメモをしているので参照。


# サードパーティ

## echo
* https://echo.labstack.com/guide/routing/
  * (IMO) もう少し具体例があると分かりやすい。
* firebase auth
  * https://gist.github.com/munierujp/e7dc02a0ec11559f9975350c9adbb065
* https://github.com/nrikiji/go-echo-sample/tree/blog-example


## go-playground/validator
* https://github.com/go-playground/validator
* requiredの場合、デフォルト値（intなら0、stringなら""）をエラーとするので注意。
  * 値として渡されたものが、デフォルトなのか渡されたものなのか区別がつかない。
  * ゼロや""を使いたい場合は、ポインタにする。
    * https://stackoverflow.com/questions/66632787/how-to-allow-zero0-value




# サンプルコード
```go
package main

import (
	"bytes"
	"context"
	"io"
	"log"
	"net/http"
	"net/http/httptest"
	"net/url"
	"strings"
	"testing"
	"time"

	"github.com/stretchr/testify/assert"
)

// http.Handlerを実装している任意のtypeは、http.Handleへ渡すことができる。
type MyHandler string

func (s MyHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	param := strings.TrimPrefix(r.URL.Path, "/hoge/")
	query := r.URL.Query().Get("q")
	vals := []string{}
	if param != "" {
		vals = append(vals, param)
	}
	if query != "" {
		vals = append(vals, query)
	}
	str := strings.Join(vals, " ")
	io.WriteString(w, str)
	w.WriteHeader(http.StatusOK)
}

// リッスン状態にしないで、ServeHTTP関数だけをテスト。
func TestSampleServerHTTP(t *testing.T) {
	server := MyHandler("")

	t.Run("test sample ServeHTTP function", func(t *testing.T) {
		request, _ := http.NewRequest(http.MethodGet, "/hoge/fuge", nil)
		response := httptest.NewRecorder()
		server.ServeHTTP(response, request)
		if response.Body.String() != "fuge" {
			t.Errorf("got %q, want %q", response.Body.String(), "fuge")
		}
	})
}

func TestSampleServer(t *testing.T) {
	// ハンドラー関数を定義する
	handler1 := func(w http.ResponseWriter, r *http.Request) {
		// wは io.Writerを実装している。
		log.Printf("%v", r)
		w.WriteHeader(http.StatusOK)

		switch r.Method {
		case http.MethodPost:
			io.WriteString(w, "post method")
		case http.MethodGet:
			io.WriteString(w, "get method")
		}
	}
	handler2 := func(w http.ResponseWriter, _ *http.Request) {
		io.WriteString(w, "bar!!\n")
		w.WriteHeader(http.StatusOK)
	}

	// http.HandleFuncを使って、パスとハンドラー関数を結びつける。
	// この関数はグローバルに定義されているマルチプレクサー（ServeMux構造体。map形式でハンドラを保有。ルーティング情報を保有。）にregisterされる。
	// このマルチプレクサーは、http.Server構造体のHandlerがnilの場合、採用されるようだ。(ref: ListenAndServe関数のコメント）
	http.HandleFunc("/foo", handler1)
	http.HandleFunc("/bar", handler2)
	http.HandleFunc("/panic", func(w http.ResponseWriter, _ *http.Request) { // panicテスト用
		panic("test")
	})

	// http.Handlerを実装している（ServeHTTP関数を実装している）任意のtypeを渡すこともできる。
	// /hoge/ としていることで、/hoge/helloなどを渡せる。
	http.Handle("/hoge/", MyHandler("Hello World."))
	// なお、http/netパッケージで定義されているtype HandlerFunc func(ResponseWriter, *Request) を指定することもできる。
	// HandlerFunc.ServeHTTPでは、そのまま自身を実行している。
	// http.Handle("/hoge/", http.HandlerFunc(func(writer http.ResponseWriter, request *http.Request){・・・}))

	// なお、自分でマルチプレクサーを作成（http.NewServeMux()）して、http.Server.Handlerへ指定する事もできる。
	// その場合は、作成した*http.ServeMuxのHandleFuncやHandleを使ってハンドラを設定する。
	// mux := http.NewServeMux()
	// mux.Handle("/", http.HandlerFunc(recoverHandler))
	// srv := &http.Server{Addr: "localhost:8080", Handler: mux}

	go func() {
		// ListenAndServeは、TCPネットワーク上で指定のアドレスにてリッスンし、コネクションの度に新たなgoルーティンを立ち上げる。
		//
		// 以下の実行と同じ。
		//(&http.Server{Addr: "0.0.0.0:8087", Handler: nil}).ListenAndServe()

		log.Fatal(http.ListenAndServe("0.0.0.0:8087", nil))
	}()

	time.Sleep(time.Microsecond * 500)

	t.Run("assert_success_HttpGet", func(t *testing.T) {
		r, err := HttpGet(MakeURL("0.0.0.0", "8087", "/foo", nil), "")
		if err != nil {
			log.Fatal("http failed:", err)
		}
		if r != "get method" {
			t.Errorf("got %q, want %q", r, "get method")
		}

		r, err = HttpGet(MakeURL("0.0.0.0", "8087", "/hoge/hello", map[string]string{"q": "world"}), "")
		if err != nil {
			log.Fatal("http failed:", err)
		}
		want := "hello world"
		if r != want {
			t.Errorf("got %q, want %q", r, want)
		}
	})

	t.Run("assert_HttpPost", func(t *testing.T) {
		r, err := HttpPost(MakeURL("0.0.0.0", "8087", "/foo", nil), `body`, "dummy token")
		assert.Equal(t, r, "post method")
		assert.Equal(t, err, nil)

		_, err = HttpPost(MakeURL("aaaa", "8087", "/foo", nil), `body`, "")
		assert.NotEqual(t, err, nil)
		_, err = HttpPost(MakeURL("////", "8087", "/foo", nil), `body`, "")
		assert.NotEqual(t, err, nil)
	})

	t.Run("assert_panic_is_recover_http_get", func(t *testing.T) {
		// panicが発生してもサーバーが落ちないことの確認。
		HttpGet(MakeURL("0.0.0.0", "8087", "/panic", nil), "")
		HttpGet(MakeURL("0.0.0.0", "8087", "/panic", nil), "")
		HttpGet(MakeURL("0.0.0.0", "8087", "/panic", nil), "")
		_, err := HttpGet(MakeURL("0.0.0.0", "8087", "/panic", nil), "")
		if err == nil {
			t.Error("err should not be null")
		}
	})
}

func MakeURL(host string, port string, path string, query map[string]string) string {
	u := &url.URL{}
	u.Scheme = "http"
	u.Host = host + ":" + port
	u.Path = path

	q := u.Query()
	for key, val := range query {
		q.Set(key, val)
	}
	u.RawQuery = q.Encode()

	return u.String()
}

type HttpRequestType int

const (
	_ HttpRequestType = iota
	HttpTypeGet
	HttpTypePost
)

func HttpDo(url string, contentType string, requestType HttpRequestType, token string, body string) (string, error) {
	// タイムアウトを30秒に指定してClient構造体を生成
	client := &http.Client{Timeout: time.Duration(30) * time.Second}

	var req *http.Request
	var err error
	if requestType == HttpTypeGet {
		req, err = http.NewRequest(http.MethodGet, url, nil)
	} else if requestType == HttpTypePost {
		var buf = bytes.NewBuffer([]byte(body)) //`{"userId": 999, "title": "try-golang"}`
		req, err = http.NewRequest(http.MethodPost, url, buf)
	}
	if err != nil {
		return "", err
	}

	// ヘッダ情報
	if contentType != "" {
		req.Header.Add("Content-Type", contentType)
	}
	if token != "" {
		req.Header.Add("authorization", "Bearer "+token)
	}

	rsp, err := client.Do(req)
	if err != nil {
		return "", err
	}

	// 関数を抜ける際に必ずresponseをcloseするようにdeferでcloseを呼ぶ。（closeしないとTCPコネクションを開きっぱなしになる。）
	defer rsp.Body.Close()

	// ステータスコードのハンドリング
	/*if rsp.StatusCode != http.StatusOK {
		logger.Error("Error Response: %s, %s", rsp.Status, rsp.Body)
		return "", err
	}*/

	// レスポンスを取得
	resBody, err := io.ReadAll(rsp.Body)
	if err != nil {
		// エラーハンドリング(省略)
		panic(err)
	}
	
	return string(resBody), nil
}

func HttpGet(url string, token string) (string, error) {
	return HttpDo(url, "application/json", HttpTypeGet, token, "")
}

func HttpPost(url string, body string, token string) (string, error) {
	return HttpDo(url, "application/json", HttpTypePost, token, body)
}

// マルチプレクサーを自分で生成
func TestSampleServerMux(t *testing.T) {
	router := http.NewServeMux()
	router.Handle("/hoge", http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		w.WriteHeader(http.StatusOK)
	}))

	t.Run("test sample ServeHTTP function", func(t *testing.T) {
		request, _ := http.NewRequest(http.MethodGet, "/hoge", nil)
		response := httptest.NewRecorder()
		router.ServeHTTP(response, request)
		if response.Result().Status != "200 OK" {
			t.Errorf("got %q, want %s", response.Result().Status, "200 OK")
		}
	})
}

func TestSampleServerGracefulShutdown(t *testing.T) {

	// シグナルによるシャットダウンのテスト
	// このテストを動かす場合はコメントアウトを取る。
	/* ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGTERM, os.Interrupt, os.Kill)
	defer stop()*/

	srv := &http.Server{
		Addr: "0.0.0.0:8087",
	}

	go srv.ListenAndServe()

	// シグナルによるシャットダウンのテスト
	// OSからのシグナルが来るまで待機する
	//<-ctx.Done()

	// 起動するまでにタイムラグがあるため待機
	time.Sleep(time.Millisecond * 100)

	// シャットダウンが完了する前に、ctxのタイマーの時刻が到来する(ctx.Done()を受信)とhttp.Server.Shutdownはエラーを返す。
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)

	// 最終的にctxをcloseする。(タイマーが到来して既にcloseしている場合はclose処理はしない)
	//
	// この処理はリソース開放も行っているため、既にcloseしているかどうかに関わらず呼び出すことが推奨されている。
	// https://pkg.go.dev/context#example-WithDeadline
	defer cancel()

	err := srv.Shutdown(ctx)
	if err != nil {
		log.Fatal("shutdown failed")
	}
	log.Print("shut down success")
}

```