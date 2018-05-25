# Go言語でcookieを扱う

## HTTPレスポンスにCookieを設定する
Goで書いたサーバ側で、HTTPレスポンスヘッダのSet-Cookieフィールドを記述する。
コードは以下。

``` go
        // 1
     cookie := &http.Cookie{
          Name: "hoge", // ここにcookieの名前を記述
          Value: "bar", // ここにcookieの値を記述
     }
        // 2
     http.SetCookie(w, cookie)
```

手順は、以下の通り
1. httpパッケージのtype Cookieのフィールドを設定する
2. httpパッケージのsetCookie関数の引数にResponseWriterとCookieを渡す
-> HTTPレスポンスヘッダのSet-CookieにCookieで追加したものが設定される

なお、NameやValue以外にもrfc6265に準拠した様々なフィールドを設定できる
cookieで設定できるフィールドは、以下のhttpパッケージのtype Cookieの限りである(公式より引用)


```
type Cookie struct {
        Name  string
        Value string

        Path       string    // optional
        Domain     string    // optional
        Expires    time.Time // optional
        RawExpires string    // for reading cookies only

        // MaxAge=0 means no 'Max-Age' attribute specified.
        // MaxAge<0 means delete cookie now, equivalently 'Max-Age: 0'
        // MaxAge>0 means Max-Age attribute present and given in seconds
        MaxAge   int
        Secure   bool
        HttpOnly bool
        Raw      string
        Unparsed []string // Raw text of unparsed attribute-value pairs
}
```
(引用元 https://golang.org/pkg/net/http/#Cookie)

## HTTPリクエストからCookieを取得する
Goで書いたサーバ側で、クライアントからのHTTリクエストのヘッダに記述されているSet-Cookieフィールドを取得する。
コードは以下。

``` go
     // 1
     cookie, err := r.Cookie("hoge")

     if err != nil {
          log.Fatal("Cookie: ", err)
     }
        // 2
     v := cookie.Value
        fmt.Println(v)

```

1. httpパッケージのtype RequestのCookieメソッドを使用して、HTTPリクエストからのcookieを取得する
2. 取得したcookieのフィールドにアクセスする


## 実装してみる
実際にGoでcookieの設定と取得を行ってみる。



``` go:main.go
package main

import (
     "net/http"
     "fmt"
     "log"
     "html/template"
)

// cookieの設定を行う
func setCookies(w http.ResponseWriter, r *http.Request) {
     cookie := &http.Cookie{
          Name: "hoge",
          Value: "bar",
     }
     http.SetCookie(w, cookie)

     fmt.Fprintf(w, "Cookieの設定ができたよ")
}

// cookieの取得を行い、htmlを返す
func showCookie(w http.ResponseWriter, r *http.Request) {
     cookie, err := r.Cookie("hoge")

     if err != nil {
          log.Fatal("Cookie: ", err)
     }

     tmpl := template.Must(template.ParseFiles("./cookie.html"))
     tmpl.Execute(w, cookie)

}

// メイン
func main() {
     http.HandleFunc("/", setCookies)
     http.HandleFunc("/cookie", showCookie)

     if err := http.ListenAndServe(":8080", nil); err != nil {
          log.Fatal("ListenAndServe: ", err)
     }
}
```

テンプレートは以下の通り

cookie.html

``` html:cookie.html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>cookie</title>
</head>

<body>
    <p>cookieの値は</p>
    <p> name: {{.Name}} </p>
    <p> value:{{.Value}} </p>
</body>

</html>
```

## cookieの確認の仕方
蛇足だが、chromeのdeveloper toolsのApplication-> Cookiesでcookieを確認できる


## 参考にさせていただいたサイト
https://golang.org/pkg/net/http/

※ ブログでも同一記事を投稿している
http://www.sekky0905.com/entry/2017/03/20/Go%E8%A8%80%E8%AA%9E%E3%81%A7Cookie%E3%82%92%E6%93%8D%E4%BD%9C%E3%81%99%E3%82%8B%28%E8%A8%AD%E5%AE%9A%E3%81%A8%E5%8F%96%E5%BE%97%29
