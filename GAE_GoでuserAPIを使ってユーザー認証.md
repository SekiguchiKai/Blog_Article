# GAE/Goでuser APIを使ってユーザー認証をする

## user APIを使うと何ができるのか

公式のAPIドキュメントを見ると以下のように記載されている
> The Users API allows an application to:

> * Detect whether the current user has signed in.
> * Redirect the user to the appropriate sign-in page to sign in.
> * Request that your application user create a new Google account if they don't have one already.

以下の公式ドキュメントから引用
https://cloud.google.com/appengine/docs/standard/go/users/

上記を意訳してみた(間違っていたら、ご指摘してください)
* 現在のユーザーが既にサインイン済みか確認する
* ユーザーをGoogleアカウントの適切なサインインページにリダイレクトさせることができる
* ユーザーがまだGoogleアカウントを取得していない場合は、Googleアカウントを取得するように要求することができる


## 実装してみた
公式のドキュメントとAPIリファレンスにしたがい、実装してみた

``` go
package main

import (
	"net/http"

	"google.golang.org/appengine"
	"google.golang.org/appengine/user"
	"html/template"
	"fmt"
)
// エントリポイント
func init() {
	http.HandleFunc("/", Entry)
	http.HandleFunc("/loggedin", LoggedIn)
}

func Entry(w http.ResponseWriter, r *http.Request) {
	// コンテキスト生成
	c := appengine.NewContext(r)

	// ログイン用URL
	// ログイン後ユーザーを引数で渡したURLへリダイレクト
	_, err := user.LoginURL(c, "/loggined")

	// エラーハンドリング
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
	}

	// テンプレートを表示
	tmpl := template.Must(template.New("enrty").Parse(entyTmpl))
	tmpl.Execute(w, nil)
}

// エントリーページのテンプレート
var entyTmpl = `
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>ログイン前</title>
</head>
<body>
   <a href="/loggedin">ログインしてね</a>
</body>
</html>
`

// ログイン後のハンドラ
func LoggedIn(w http.ResponseWriter, r *http.Request) {
	// コンテキスト生成
	c := appengine.NewContext(r)

	// 現在ログインしているユーザーの情報を取得する
	u := user.Current(c)

	// 現在ログインしているユーザーがいない場合
	if u == nil {
		// ユーザーにサインインするように促すためのページのURLを返す
		// ログイン後ユーザーを引数で与えたURLにリダイレクトさせる
		url, _ := user.LoginURL(c, "/")
		fmt.Fprintf(w, `<a href="%s">ログイン用のサイトに移る</a>`, url)
		return
	}

	// ログアウト用のURL
	// ログインアウト後ユーザーを引数で渡したURLへリダイレクト
	_, err := user.LogoutURL(c, "/")

	// エラーハンドリング
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
	}

	// テンプレートを表示
	tmpl := template.Must(template.New("loggedIn").Parse(loggedInTmpl))
	tmpl.Execute(w, nil)
}

// ログイン後のページのテンプレート
var loggedInTmpl = `
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>ログイン後</title>
</head>
<body>
   <p>ログインできたよ</p>
   <a href="/">ログアウト</a>
</body>
</html>
`
```

## サイトの遷移
サイトの遷移は、大まかに言うと以下のような流れだ<br>
1. リクエストを送ると実装したアプリケーションのページがレスポンスとして返って来る
->これは、``` go 	http.HandleFunc("/", Entry) ``` で登録したハンドラが発動している

2. ``` go http.HandleFunc("/", Entry) ``` のハンドラによって表示されたHTMLページの「ログインしてね」をクリックする

3. 「ログインしてね」をクリックすると ``` go http.HandleFunc("/loggedin", LoggedIn) ``` で登録したハンドラが発動する

4. ログインしていない場合は、 以下の部分が発動して、『ログイン用のサイトに移る』という表示が現れる

``` go	
        // 現在ログインしているユーザーの情報を取得する
	u := user.Current(c)

	// 現在ログインしているユーザーがいない場合
	if u == nil {
		// ユーザーにサインインするように促すためのページのURLを返す
		// ログイン後ユーザーを引数で与えたURLにリダイレクトさせる
		url, _ := user.LoginURL(c, "/")
		fmt.Fprintf(w, `<a href="%s">ログイン用のサイトに移る</a>`, url)
		return
	} 
```

5. 『ログイン用のサイトに移る』をクリックするとGoogleアカウントの認証用のページ(外部のGoogleのサイト)にリダイレクトする

6. リダイレクト先の認証用のページで認証が完了したら、ログインが完了し ```go http.HandleFunc("/loggedin", LoggedIn) ``` 内のHTMLテンプレートが表示される（「ログインできたよ」が表示される）


以下の公式のドキュメントがわかりやすい
https://cloud.google.com/appengine/docs/standard/go/users/
https://cloud.google.com/appengine/docs/standard/go/users/reference

## 参考にさせていただいたサイト
user APIについて<br>
https://cloud.google.com/appengine/docs/standard/go/users/
https://cloud.google.com/appengine/docs/standard/go/users/reference
Basic認証について<br>
http://qiita.com/r7kamura/items/69904137ea20b6b86822
IPAのOAuth 2.0の説明<br>
https://www.ipa.go.jp/security/awareness/vendor/programmingv2/contents/709.html

## 参考文献
茨木 隆彰　(2012/02)『はじめてのGoogle App Engine Go言語編 (I・O BOOKS)』 工学社

※ ブログでも同一記事を投稿している
http://www.sekky0905.com/entry/2017/02/23/GAE/Go%E3%81%A7user_API%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E3%83%A6%E3%83%BC%E3%82%B6%E3%83%BC%E8%AA%8D%E8%A8%BC%E3%82%92%E3%81%99%E3%82%8B
