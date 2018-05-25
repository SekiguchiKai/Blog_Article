# GoのTemplateのPipelines(パイプライン)を理解しながら、テンプレート内で関数にスライスを引数で渡してみる

# Goのテンプレートの中で関数を使い、その関数に引数を渡す
GoでMVCなアプリを書くときに重宝するであろう、templateパッケージ<br>

そのtemplateパッケージを使用してGoでテンプレートを書き、その中で関数を呼び出し、引数に構造体のフィールド上に存在するスライスの要素を渡す<br>

GoのtemplateのPipelinesを理解するのにも役に立つと思うのでメモ<br>

## GoのTemplateでスライスを扱うには
Goのtemplateのあらかじめ定義されたグローバル関数であるindexを使用する<br>

indexは
>最初の引数を続く引数でインデックスした結果を返します。
例えば"index x 1 2 3"はGoの文法では x[1][2][3]となります。
インデックスされる値はマップ、スライス、配列である必要があります。
(引用元:http://golang-jp.org/pkg/text/template/)

## GoののTemplateで関数を扱うには
FuncMapで関数の名前と関数のmapを定義する<br>
詳しくは、後述のコードを参照<br>


## GoのPipelines(パイプライン)とは
>パイプラインはパイプライン文字'|'で区切ってコマンドをつなげることによって「連鎖」させることができます。 連鎖されたパイプラインでは、各コマンドの結果は次のコマンドへ最後の引数として渡されます。 パイプラインにおける最後のコマンドの出力がそのパイプラインの値になります。
(引用元: http://golang-jp.org/pkg/text/template/)

例えば、aというコマンドとbというコマンドが存在した場合、<br>

```
a | b
```

というコードは、Goでは、aのコマンドの実行結果が、bのコマンドの最後の引数となる<br>

## 実装してみる

```go

package main

import (
	"html/template"
	"net/http"
	"log"
)
// メイン関数
func main() {
	// ハンドラをバンドル
	http.HandleFunc("/", Handler)

	// リスナー
	if err := http.ListenAndServe(":8080", nil); err != nil {
		log.Fatal("ListenAndServe: ", err)
	}
}

// テンプレートを生成
func Handler(w http.ResponseWriter, r *http.Request) {
	// 構造体Programを生成
	p := &Program{Languages:[]string{"Go", "Java", "Python", "JavaScript"},}

	// 関数を名前をつけてtemplateのFuncMapに登録
	funcMap := template.FuncMap{
		"judgeLangType" : JudgeLangType,
	}

	// テンプレート部分
	text := `
	Go : {{ index .Languages 0 | judgeLangType}}
	Java : {{ index .Languages 1 | judgeLangType}}
	Python : {{ index .Languages 2 | judgeLangType}}
	JavaScript : {{ index .Languages 3 | judgeLangType}}
	`

	// テンプレートをパースして、http.ResponseWriterに書き込み
	tmpl := template.Must(template.New("calculator").Funcs(funcMap).Parse(text))
	tmpl.Execute(w, p)
}

// 構造体
type Program struct {
	Languages []string
}

// 言語を引数に受けて、その言語の型を返す
func JudgeLangType(lang string) string{
	switch lang {
	case "Go", "Java":
		return "Static"
	case "Python", "JavaScript":
		return "Dynamic"
	default:
		return "unknown"
	}
}

```

実行結果

```
	Go : Static
	Java : Static
	Python : Dynamic
	JavaScript : Dynamic
 
```



上記のコードのテンプレート部分である以下に注目する

``` go
	// テンプレート部分
	text := `
	Go : {{ index .Languages 0 | judgeLangType}}
	Java : {{ index .Languages 1 | judgeLangType}}
	Python : {{ index .Languages 2 | judgeLangType}}
	JavaScript : {{ index .Languages 3 | judgeLangType}}
	`
```

```
Go : {{ index .Languages 0 | judgeLangType}}
```

* ``` index .Languages 0```  で、構造体 ``` Program```  のフィールド ``` Language```  のスライスの0番目の要素を表している
* ``` | ```  で、パイプラインを表している
* ``` judgeLangType```  は、``` template.FuncMap```  で登録した関数を表している

これは、``` |```  の左側の ``` index .Languages 0```  の結果(Languagesスライスの0番目の要素)が ``` |```の右側の ```  judgeLangType```  関数の引数になっている

## 参考にさせていただいたサイト
http://golang-jp.org/pkg/text/template/#FuncMap
https://gohugo.io/templates/go-templates/

※ ブログでも同一記事を投稿している
http://www.sekky0905.com/entry/2017/03/22/Go%E3%81%AETemplate%E3%81%AEPipelines%28%E3%83%91%E3%82%A4%E3%83%97%E3%83%A9%E3%82%A4%E3%83%B3%29%E3%82%92%E7%90%86%E8%A7%A3%E3%81%97%E3%81%AA%E3%81%8C%E3%82%89%E3%80%81%E3%83%86%E3%83%B3%E3%83%97%E3%83%AC


