# GoのTemplateパッケージを使い作成したHTMLテンプレートでCSS等の静的ファイルをリンクさせる時の注意
Goでtemplateパッケージを使用し、MVCなアプリを作成した時に、ちょっと引っかかったので、メモ<br>
Goでwebサーバを立て、Templateパッケージを使い作成したHTMLテンプレートを表示させたところ、HTMLテンプレートでリンクしたcssが適用されないという事態に陥った<br>

ダメなコードと、解決済みのコードを使ってメモする<br>

以下の簡単なパッケージ構成のコードを例にする<br>

``` 
sample
      |_resources <-以下に各種類別静的ファイル(今回はcssのみ)
      |         |_css
      |_view
      |    |_index.html(テンプレート)
      |_main.go
      
``` 

なお、HTMLテンプレートとCSSは以下を例にする<br>


index.html

``` html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>テスト</title>
    <link rel="stylesheet" type="text/css" href="../resources/css/index.css">
</head>

<body>
<p>名前:{{.Name}}</p>
<p>出身:{{.From}}</p>
</body>

</html>
```

index.css

``` css
p {
color:blue;
}
```


## HTMLのテンプレートにCSSが反映されないコード
以下のコードではHTMLのテンプレートにCSSが反映されない

``` go
package main

import (
	"fmt"
	"net/http"
	"html/template"
)

func main() {
	fmt.Println("The Server runs with http://localhost:8080/")

	http.HandleFunc("/", Handler)
	http.ListenAndServe(":8080", nil)
}

type Person struct {
	Name string
	From string
}

func Handler(w http.ResponseWriter, r *http.Request) {

	p := Person{
		Name:"sekky",
		From : "埼玉",
	}

	tmpl := template.Must(template.ParseFiles("./view/index.html"))
	tmpl.Execute(w, p)
} 
```

## HTMLのテンプレートにCSSが反映されるコード

以下のコードでは、HTMLのテンプレートにCSSが反映される

``` go
package main

import (
	"fmt"
	"net/http"
	"html/template"
)

func main() {
	fmt.Println("The Server runs with http://localhost:8080/")
	// ここで静的ファイルであるCSSを適用
	// Handler : 第一引数で与えたパターンに対して、第二引数Handlerを登録
	// StripPrefix : 第一引数で与えたURLのパスを取り除いて、第二引数Handlerを発動させる
	// FileServer : 、HTTPリクエストに対して、第一引数のrootを起点とするファイルシステムのコンテンツを返すハンドラを返す
	// ここでいうrootは、resources

	// つまり、resourcesディレクトリ以下の静的ファイル(ここでいうcss/index.css)を探して返す
	http.Handle("/resources/", http.StripPrefix("/resources/", http.FileServer(http.Dir("resources/"))))
	http.HandleFunc("/", Handler)
	http.ListenAndServe(":8080", nil)
}

type Person struct {
	Name string
	From string
}

func Handler(w http.ResponseWriter, r *http.Request) {

	p := Person{
		Name:"sekky",
		From : "埼玉",
	}

	tmpl := template.Must(template.ParseFiles("./view/index.html"))
	tmpl.Execute(w, p)
}
```

## どこが違うのか
HTMLのテンプレートにCSSが反映されるコードと反映されないコードは、以下の1行が存在するかしないかの差

``` go 
http.Handle("/resources/", http.StripPrefix("/resources/", http.FileServer(http.Dir("resources/"))))
``` 

## この一行で何をやっているのか
使用しているhttpパッケージの各メソッドの説明はコード内のコメントに記載したが、ざっくり言うとあるディレクトリを指定して、そのディレクトリから静的ファイル（ここでいうcssファイル）をレスポンスするようにハンドリングしている

個々のメソッドについては、公式のドキュメントを見るとわかりやすい
https://golang.org/pkg/net/http/

## 参考にさせていただいたサイト
http://stackoverflow.com/questions/13302020/rendering-css-in-a-go-web-application
https://golang.org/pkg/net/http/

※ ブログでも同一記事を投稿している
http://www.sekky0905.com/entry/2017/02/23/_Go%E8%A8%80%E8%AA%9E%E3%81%AETemplate%E3%83%91%E3%83%83%E3%82%B1%E3%83%BC%E3%82%B8%E3%82%92%E4%BD%BF%E3%81%84%E4%BD%9C%E6%88%90%E3%81%97%E3%81%9FHTML%E3%83%86%E3%83%B3%E3%83%97%E3%83%AC%E3%83%BC%E3%83%88
