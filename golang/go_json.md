---
title: "json - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > json


# json.Unmarshal
* json.Unmarshalの第二引数へ、interface{}型や、map[string]interface{}型を渡すことで、Mapに変換したものを入れてくれる。
* 構造体を渡すと、構造体へ変換してくれる。
```go
import (
  "encoding/json"
  "fmt"
)
func main() {
  data := `{"name":"hoge", "age":28}`
  mapData := map[string]interface{}{}
  json.Unmarshal([]byte(data), &mapData)
  fmt.Println(mapData["name"].(string))
}
```
# 構造体への変換について注意。
* json -> 構造体の変換は構造体自体を宣言しておく必要がある。
* ネストされた構造体にany型が入っているとmapになってしまうので注意。
  * anyのフィールドへ別の構造体（json宣言がされているもの）を入れても mapになってしまう。
  * １時間くらいハマった。
* なお、json.Decoderを使った場合も同じ結果だった。
```go
package main

import (
	"encoding/json"
	"fmt"
)

func main() {
	data := `{
			"result":"success", 
			"data": {
				"entity": "aaa"
			}
		}`

  // これだと、Dataフィールドがany型になっていて、たとえそこに具体的な構造体を代入しても、mapとして取り込みされてしまう。
	respObj1 := struct {
		Result string `json:"result"`
		Data   any    `json:"data"`// ここがany型になっているのでmapとして取り込みされてしまう。
	}{
		Data: struct {
			Entity string `json:"entity"`// リテラル内で別の構造体として入れてもmapになってしまう。
		}{},
	}
	json.Unmarshal([]byte(data), &respObj1)
	fmt.Printf("%+v \n", respObj1) // {Result:success Data:map[entity:aaa]}

  // これにするとすべて構造体へ取り込みされる。
	respObj2 := struct {
		Result string `json:"result"`
		Data   struct {
			Entity string `json:"entity"`
		} `json:"data"`
	}{}

	json.Unmarshal([]byte(data), &respObj2)
	fmt.Printf("%+v \n", respObj2) // {Result:success Data:{Entity:aaa}}
}
```
# Unmarshalの注意
* 対象のjsonに指定した構造体が入って無くてもエラーにならない。（デフォルト値が入る）
* 対象のjsonに指定した構造体以外が入っていても無視される。
## (IME) json.Unmarshalのエラーハンドリングの注意
* フィールド内で例えば、uuidなどのデコードに失敗した場合でも、特にエラーのラップもせずにそのまま返ってくる
## 配列を含むjsonを、mapへunmarshalする
* []mapにする必要がある。structにする場合も同様に配列にする必要がある。
* そうしないと、json: cannot unmarshal array into Go value　〜ってエラーが出る
https://stackoverflow.com/questions/25465566/golang-parse-json-array-into-data-structure
https://stackoverflow.com/questions/42289591/unmarshaling-json-top-level-array-into-map-of-string-to-string




# json.Marshal
```go
import (
	"encoding/json"
	"fmt"
)

type s1 struct {
	F1 int    `json:"f1"`
	F2 string `json:"f2"`
}

func main() {
	v := s1{F1: 3, F2: "aaaa"}
	j, _ := json.Marshal(&v)// 戻り値は[]byte型。
	fmt.Println("%+v", string(j)) 
}
```
## omitemptyや-にフィールドを対象外とすることができる。
* https://pkg.go.dev/encoding/json#Marshal
> omitempty "オプションは、フィールドが空の値（false、0、nilポインタ、nilインターフェース値、長さ0の配列、スライス、マップ、文字列として定義される）を持つ場合、そのフィールドをエンコードから省略することを指定する。 特別なケースとして、フィールドタグが"-"の場合、フィールドは常に省略される。 名前"-"のフィールドは、タグ"-, "を使っても生成できることに注意。
* これらはUnmarshalでは影響しない点に注意。




## json.Encoder、json.Decoderを使う方法もある。
* json.Marshalと違うのは、io.Writerへ渡す形式であること。
	* なのでファイルへの書き込みでも、バッファへの出力、標準出力への出力など汎用的にできる感じ。
```go
type Person struct {
    Name string
    Age  int
}

...

jsonReader := strings.NewReader(`{"Name":"Alice","Age":20}`)
jsonDecoder := json.NewDecoder(jsonReader)
var p Person
err := jsonDecoder.Decode(&p)
if err != nil {
	log.Fatal(err)
}
fmt.Println(p)
```
```go
v := s1{F1: 3, F2: "aaaa"}
buffer := &bytes.Buffer{}
err := json.NewEncoder(buffer).Encode(v)
// err := json.NewEncoder(os.Stdout).Encode(stcData)// 標準出力の場合
if err != nil {
	fmt.Println(err)
}
fmt.Println(buffer.String())
```
* https://qiita.com/hrk_ym/items/41175f4a902a3fcb2896
* https://zenn.dev/hsaki/articles/go-convert-json-struct
* https://otameshi61.hatenablog.com/entry/2022/08/11/100322
* https://tech.yappli.io/entry/go_unmarshal_interface


# 戻り値のjsonのテスト
* jsonをUnmarshalしてMapにする。同じく想定値をjsonとして用意してそれをMapにして両者をDeep Equalで比較。
* https://handlename.hatenablog.jp/entry/2014/10/02/100444



# Jsonの文字列はU+000A （"\n"で表現されるunicode） を含んではいけない。
* Jsonの仕様として、含むことは禁止されている。
* そして、Goでもencoding/jsonでパースするときもこの仕様に従っている。
https://blog.nishimu.land/entry/2014/04/16/213243
https://blog.kyanny.me/entry/2020/08/26/001233
```go
var str string
if err := json.Unmarshal([]byte("\"\n\""), &str); err != nil {
    fmt.Print(err)//  &json.SyntaxError{msg:"invalid character '\\n' in string literal",...}
}
```



# テストコード
```go
import (
	"encoding/json"
	"testing"
	"time"

	"github.com/stretchr/testify/assert"

	"github.com/google/uuid"
)

func GetFirst[A any](a A, b ...any) A {
	return a
}

func GetLast(a ...any) any {
	return a[len(a)-1]
}

func UnmarshalString[S any](jsn string, to *S) (*S, error) {
	if err := json.Unmarshal([]byte(jsn), to); err != nil {
		return nil, err
	}
	return to, nil
}

func Ptr[T any](a T) *T {
	return &a
}

func TestJson(t *testing.T) {
	t.Run("assert_json", func(t *testing.T) {
		assert.Exactly(t, string(GetFirst(json.Marshal("aaaaaa"))), `"aaaaaa"`)

		assert.Exactly(t, *GetFirst(UnmarshalString(`{"number":1}`, &struct {
			Number *int `json:"number"`
		}{})), struct {
			Number *int `json:"number"`
		}{
			Number: Ptr(1),
		})

		assert.Exactly(t, *GetFirst(UnmarshalString(`{"number":0}`, &struct {
			Number *int `json:"number"`
		}{})), struct {
			Number *int `json:"number"`
		}{
			Number: Ptr(0),
		})

		assert.Exactly(t, *GetFirst(UnmarshalString(`{}`, &struct {
			Number *int `json:"number"`
		}{})), struct {
			Number *int `json:"number"`
		}{
			Number: nil,
		})

		assert.Exactly(t, *GetFirst(UnmarshalString(`{"created_at":"2023-05-04T00:00:00.000000Z"}`, &struct {
			CreatedAt *time.Time `json:"created_at"`
		}{})), struct {
			CreatedAt *time.Time `json:"created_at"`
		}{
			CreatedAt: Ptr(GetFirst(time.Parse("2006-01-02T15:04:05.000000Z07:00", "2023-05-04T00:00:00.000000Z"))),
		})

		assert.Exactly(t, *GetFirst(UnmarshalString(`{}`, &struct {
			CreatedAt *time.Time `json:"created_at"`
		}{})), struct {
			CreatedAt *time.Time `json:"created_at"`
		}{
			CreatedAt: nil,
		})

		// json側がnullの場合は ゼロ値として値が入る。
		assert.Exactly(t, GetFirst(UnmarshalString(`{"ID":null}`, &struct{ ID string }{})), &struct{ ID string }{})
		assert.Exactly(t, GetFirst(UnmarshalString(`{"ID":null}`, &struct{ ID time.Time }{})), &struct{ ID time.Time }{})
		assert.Exactly(t, *GetFirst(UnmarshalString(`null`, Ptr(""))), "")
		assert.Exactly(t, GetLast(UnmarshalString(`{"ID":3}`, &struct{ ID int }{})), nil)
		assert.NotEqual(t, GetLast(UnmarshalString(`{"ID":"aaaa"}`, &struct{ ID int }{})), nil)
		assert.Exactly(t, GetLast(UnmarshalString(`{"ID":3}`, &struct{ ID uint }{})), nil)

		// json側が空文字の場合は、型が一致しないとエラーになる。
		assert.Exactly(t, GetFirst(UnmarshalString(`{"ID":""}`, &struct{ ID string }{})), &struct{ ID string }{})
		assert.NotEqual(t, GetLast(UnmarshalString(`{"ID":""}`, &struct{ ID int }{})), nil)
		assert.NotEqual(t, GetLast(UnmarshalString(`{"ID":""}`, &struct{ ID time.Time }{})), nil) // 日時としてparseできずにエラー
		assert.NotEqual(t, GetLast(UnmarshalString(`""`, &time.Time{})), nil)                     // 日時としてparseできずにエラー
		/// stringの場合はOK
		assert.Exactly(t, GetLast(UnmarshalString(`{"ID":""}`, &struct{ ID string }{})), nil)
		assert.Exactly(t, *GetFirst(UnmarshalString(`""`, Ptr(""))), "")

		// 何も入っていないtime.Timeはデフォルトが0001-01-01T00:00:00Z
		assert.Exactly(t, string(GetFirst(json.Marshal(time.Time{}))), "\"0001-01-01T00:00:00Z\"")

		// uuid.UUIDのデフォルト値は00000000-0000-0000-0000-000000000000
		assert.Exactly(t, string(GetFirst(json.Marshal(uuid.UUID{}))), "\"00000000-0000-0000-0000-000000000000\"")
		assert.Exactly(t, string(GetFirst(json.Marshal(struct{ ID uuid.UUID }{}))), "{\"ID\":\"00000000-0000-0000-0000-000000000000\"}")

	})
}

```


# テストコード(エスケープシーケンス)
## まとめ
* Unmarshal(Json -> データ)の文字列に対する処理
	* エスケープシーケンスは特殊文字へ変換
	* エスケープされているエスケープ文字はその文字として変換（例えば「\\n」は「\n」として取り込む）
	* HTML文字に対応するunicodeシーケンスはHTMLへ変換
	* 一部の特殊文字（LFやCR等）、存在しないエスケープシーケンスが含まれる場合はエラーとなる。
		* スラッシュやダブルクオテーション（エスケープしたもの）はOK。
		* ダブルクオテーションはエスケープしていない場合はエラーとなる。
* Marshal(データ -> Json)の文字列に対する処理
	* 文字列中の特殊文字をエスケープシーケンスへ変換
	* エスケープシーケンスはエスケープ（例えば「\n」は「\\n」へ変換）
	* HTML文字をunicodeのシーケンスへ変換
	* ダブルクオテーションが入っている場合はエスケープ
* json.Decoder.Decode
	* Unmarshalとほぼ同じ挙動
	* ダブルクオテーションはエスケープしていない場合でもエラーとはならないが消去される。
* json.Encoder.Encode
	* Marshalとほぼ同じ挙動だが、 HTML文字の置き換えはEncoder.SetEscapeHTML(false)をしておくとOFFになる。
	* 末尾に"\n"が追加される。(これに関しては意図が理解できていない)
## IMO
* 上記の文字列の変換処理はオプションによってオプトアウトできると良いと感じた。
* 事由
	* Jsonの変換処理に、文字列の内容自体への副作用があると考慮事項が増える。
	* 上記の変換はXSSやCSRFなどのセキュリティのための考慮も含まれていると考えられる。しかし、Jsonの扱い方やセキュリティ対策はサーバーやアプリケーションによって構成が様々であるため、 Jsonの変換自体はRFCに従う限りとした方が適切
* ref ここに少し近いことが書いてある。
	* https://golang50shad.es/

## RFC
* Jsonの仕様ではダブルクオテーション、スラッシュ、バックスラッシュ、一部の特殊文字を含むことはできないとRFCに定められている。
* https://datatracker.ietf.org/doc/html/rfc8259

```go
import (
	"bytes"
	"encoding/json"
	"testing"

	"github.com/stretchr/testify/assert"
)

func GetFirst[A any](a A, b ...any) A {
	return a
}

func Ptr[T any](a T) *T {
	return &a
}

func TestJsonEscape(t *testing.T) {

	t.Run("assert_Json_escape_by_unmarshal/marshal", func(t *testing.T) {

		// json.Unmarshalの挙動は以下。
		var str string
		assert.NotNil(t, json.Unmarshal([]byte(`"""`), &str))  // これはエラー。
		assert.Nil(t, json.Unmarshal([]byte(`"\""`), &str))    // エスケープさえすればOKなようだ。
		assert.NotNil(t, json.Unmarshal([]byte(`"\"`), &str))  // \に続くエスケープシーケンスを表す文字がないためエラーになる。
		assert.NotNil(t, json.Unmarshal([]byte(`"\p"`), &str)) // 「\p」は存在しないエスケープシーケンスのためエラーになる。
		assert.Nil(t, json.Unmarshal([]byte(`"/"`), &str))     // スラッシュもエラーにならない。
		assert.Nil(t, json.Unmarshal([]byte(`"><&"`), &str))   // HTML文字が入っててもエラーにはならない。
		assert.Equal(t, str, `><&`)
		assert.Nil(t, json.Unmarshal([]byte(`"`+"\u2028"+`"`), &str))
		assert.Equal(t, str, "\u2028")
		assert.Nil(t, json.Unmarshal([]byte(`"`+"\u2029"+`"`), &str))
		assert.Equal(t, str, "\u2029")
		assert.Nil(t, json.Unmarshal([]byte(`"`+`\u003c`+`"`), &str)) // 一方、HTML文字を表すunicode文字列はHTML文字へ変換される。
		assert.Equal(t, str, `<`)
		assert.Nil(t, json.Unmarshal([]byte(`"`+`\u003e`+`"`), &str))
		assert.Equal(t, str, `>`)
		assert.Nil(t, json.Unmarshal([]byte(`"`+`\u0026`+`"`), &str))
		assert.Equal(t, str, `&`)
		assert.Nil(t, json.Unmarshal([]byte(`"`+`\u2028`+`"`), &str))
		assert.Equal(t, str, "\u2028")
		assert.Nil(t, json.Unmarshal([]byte(`"`+`\u2029`+`"`), &str))
		assert.Equal(t, str, "\u2029")
		assert.NotNil(t, json.Unmarshal([]byte(`"`+"\n"+`"`), &str)) // 特殊文字はエラー。
		assert.Equal(t, json.Valid([]byte(`"`+"\n"+`"`)), false)     // 特殊文字はJsonとしてValidではない。（cf） jqコマンドでも例えば、 echo '"\n"' | jq　のように実行するとパースエラーになる）
		assert.NotNil(t, json.Unmarshal([]byte("\"\u0000\""), &str))
		assert.NotNil(t, json.Unmarshal([]byte("\"\u0004\""), &str))
		assert.NotNil(t, json.Unmarshal([]byte("\"\u0019\""), &str))
		assert.NotNil(t, json.Unmarshal([]byte("\"\b\""), &str))
		assert.NotNil(t, json.Unmarshal([]byte("\"\t\""), &str))
		assert.NotNil(t, json.Unmarshal([]byte("\"\f\""), &str))
		assert.Nil(t, json.Unmarshal([]byte(`"\u0000\u0001\u0002\u0003\u0004\b\t\n\f\r\u0019あ"`), &str)) // エスケープシーケンスは特殊文字へ変換される。
		assert.Equal(t, str, "\u0000\u0001\u0002\u0003\u0004\b\t\n\f\r\u0019あ")
		assert.Nil(t, json.Unmarshal([]byte(`"`+`\\n\"`+`"`), &str)) // エスケープされているものはその文字として解釈される。
		assert.Equal(t, str, `\n"`)
		assert.Nil(t, json.Unmarshal([]byte(`"`+"\xe3\x81\x82"+`"`), &str))
		assert.Equal(t, str, `あ`)
		assert.Nil(t, json.Unmarshal([]byte(`"`+"\xe3\x81"+`"`), &str)) // unmarshalのコメントにあるように、UTF-8として無効な文字が含まれている場合はエラーにならずにU+FFFDへ変換される。
		assert.Equal(t, str, "\uFFFD\uFFFD")

		// json.Marshal関数はエスケープシーケンスが含まれている場合、エスケープする。
		// そして、特殊文字はエスケープシーケンスへ変換される。
		//（これはオプションなどで回避することはできない。json.NewEncoderを使えば可能。）
		// （外側の「"」はjsonの文字列として自動的に付与される）
		assert.Equal(t, string(GetFirst(json.Marshal(`\n`))), `"`+`\\n`+`"`) // 「\n」は 「\\n」に変換される。
		assert.Equal(t, string(GetFirst(json.Marshal(`"`))), `"`+`\"`+`"`)   // 「"」は 「\"」に変換される。
		assert.Equal(t, string(GetFirst(json.Marshal(`/`))), `"`+`/`+`"`)    // スラッシュはエスケープされない。
		assert.Equal(t, string(GetFirst(json.Marshal(`"\\b\t\n\f\r`))), `"`+`\"\\\\b\\t\\n\\f\\r`+`"`)
		assert.Equal(t, string(GetFirst(json.Marshal("\u0000\u0001\u0002\u0003\u0004\u0008\u0009\u000A\u000C\u000D\u0019\u0020\u0021"))), `"`+`\u0000\u0001\u0002\u0003\u0004\u0008\t\n\u000c\r\u0019 !`+`"`) // 特殊文字はエスケープシーケンスへ変換される。（\u0008, \u000Cは、 \b, \f　にはならないようだ。）
		// なお、Marshal関数のコメントにあるように、上記に加えて、
		// HTML文字（"<", ">", "&", U+2028(LS), and U+2029（PS））はunicodeの文字列へ変換される。
		assert.Equal(t, string(GetFirst(json.Marshal(`<`))), `"`+`\u003c`+`"`)
		assert.Equal(t, string(GetFirst(json.Marshal(`>`))), `"`+`\u003e`+`"`)
		assert.Equal(t, string(GetFirst(json.Marshal(`&`))), `"`+`\u0026`+`"`)
		assert.Equal(t, string(GetFirst(json.Marshal("\u2028"))), `"`+`\u2028`+`"`)
		assert.Equal(t, string(GetFirst(json.Marshal("\u2029"))), `"`+`\u2029`+`"`)
	})

	t.Run("assert_Json_escape_by_decoder/encoder", func(t *testing.T) {
		//var result map[string]interface{}
		var result string
		// Unmarshalと同様にエスケープシーケンスは特殊文字へ変換される。(回避不可能？)
		assert.Nil(t, json.NewDecoder(bytes.NewReader([]byte(`"\b\f\n\r\tあ"`))).Decode(&result))
		assert.Equal(t, result, "\b\f\n\r\tあ")
		// これはUnmarshalと違ってエラーにならない。（削除される）
		assert.Nil(t, json.NewDecoder(bytes.NewReader([]byte(`"""`))).Decode(&result))
		assert.Equal(t, result, "")
		// 以下はUnmarshalと同様にエラー
		assert.NotNil(t, json.NewDecoder(bytes.NewReader([]byte(`"\"`))).Decode(&result))
		assert.NotNil(t, json.NewDecoder(bytes.NewReader([]byte(`"\p"`))).Decode(&result))
		assert.Nil(t, json.NewDecoder(bytes.NewReader([]byte(`"><&"`))).Decode(&result))
		assert.Equal(t, result, `><&`)
		assert.NotNil(t, json.NewDecoder(bytes.NewReader([]byte("\"\n\""))).Decode(&result))
		// Unmarshalと同様にHTML文字に対応するunicode文字列をHTML文字へ変換される。
		assert.Nil(t, json.NewDecoder(bytes.NewReader([]byte(`"`+`\u003c`+`"`))).Decode(&result))
		assert.Equal(t, result, `<`)

		// Encoderは仕様として末尾に"\n"が設定される。（この挙動の事由は分からなかった。）
		// それ以外の挙動はMarshalと同じようだ。
		// https://stackoverflow.com/questions/36319918/why-does-json-encoder-add-an-extra-line
		resultB := bytes.Buffer{}
		//enc.SetEscapeHTML(false)
		assert.Nil(t, json.NewEncoder(&resultB).Encode(`\n`))
		assert.Equal(t, resultB.String(), `"`+`\\n`+`"`+"\n")
		resultB = bytes.Buffer{}
		assert.Nil(t, json.NewEncoder(&resultB).Encode("\n"))
		assert.Equal(t, resultB.String(), `"`+`\n`+`"`+"\n")
		resultB = bytes.Buffer{}
		assert.Nil(t, json.NewEncoder(&resultB).Encode(`"\\b\f\n\r\t`))
		assert.Equal(t, resultB.String(), `"`+`\"\\\\b\\f\\n\\r\\t`+`"`+"\n")
		resultB = bytes.Buffer{}
		assert.Nil(t, json.NewEncoder(&resultB).Encode(`<`))
		assert.Equal(t, resultB.String(), `"`+`\u003c`+`"`+"\n")
		// ただし、SetEscapeHTMLをfalseで設定すると、HTML文字の変換はされないようだ。
		resultB = bytes.Buffer{}
		en := json.NewEncoder(&resultB)
		en.SetEscapeHTML(false)
		assert.Nil(t, en.Encode(`<`))
		assert.Equal(t, resultB.String(), `"`+`<`+`"`+"\n")
	})
}
```