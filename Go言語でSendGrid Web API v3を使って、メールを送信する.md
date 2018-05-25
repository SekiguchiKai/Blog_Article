# Go言語でSendGrid Web API v3を使って、メールを送信する
UIやAPIを通じて簡単にメールを送信してくれるSendGrid。

ライブラリが非常に使いやすく便利で、今後使用することが多々ありそうなので、メモ。


## SendGridとは
簡単に言うと、UIやAPIを通じて、送信したいメールを設定すると、代わりにメールを送信してくれるクラウドサービス。


公式の説明は以下の通り

> SendGridは全世界で利用されているメール配信サービスです。
クラウドサービスのためアカウントを作成するだけで即日メールを
送信でき、面倒でコストのかかるメールサーバの構築は不要です。


引用元: https://sendgrid.kke.co.jp/about/


## 今回やること
ローカルのGoのコードから、SendGrid Web APIにリクエストを送信して、メールを送信してもらう。

どうやらv3でBREAKING CHANGESがあったようなので、今回はそのSendGrid Web API v3をGoで実装する。

https://github.com/sendgrid/sendgrid-go/issues/81



## 簡単な仕組みの説明
1. Goでメールの設定を記述したJSONを作成し、それをリクエストボディに詰める
2. ローカルのGoのコードから、SendGrid Web APIに対してリクエストを送信する
3. クラウドのSendGridのサービスが、送信したリクエストをjsonで設定したメールを代わりに送信してくれる


## 実装の大まかな手順
大まかな手順は以下の通り。
1. 取得したAPI KEYとエンドポイント、ホストからrestパッケージのRequestを生成
2. リクエストのメソッドを決定
3. リクエストのボディで、送信するメールのメタデータをJSONで作成する
4. 作成したリクエストを送信


とまあ、一般的なHTTP通信に則った手順で、SendGrid APIにメールを送信するためのリクエストを送信することができる。


※ なお、API KEYの取得に関しては、本題から逸れるので記述しない。
以下の日本語の公式ドキュメントを元にすれば、簡単に作成できるはずなので以下を参照されたい。
https://sendgrid.kke.co.jp/docs/User_Manual_JP/Settings/api_keys.html

##　実装してみた
goのコード

``` go
package main

import (
	"encoding/json"
	"fmt"
	"github.com/joho/godotenv"
	"github.com/sendgrid/sendgrid-go"
	"log"
	"os"
)

// requert.Bodyに格納するJSONの元となるメールを表す構造体
type Mail struct {
	Subject          string             `json:"subject"`
	Personalizations []Personalizations `json:"personalizations"`
	From             MailUser           `json:"from"`
	Content          []Contents         `json:"content"`
}

// 封筒のようなもの
// メールのメタデータを表す構造体
type Personalizations struct {
	To []MailUser `json:"to"`
}

// メールのユーザーを表す構造体
type MailUser struct {
	Email string `json:"email"`
	Name  string `json:"name"`
}

// メールの中身を表す構造体
type Contents struct {
	Type  string `json:"type"`
	Value string `json:"value"`
}

// .envファイルを読み込んで、ロードする
func Env_load() {
	// .envファイルを読み込んで、ロード
	err := godotenv.Load()
	if err != nil {
		log.Println(err)
	}
}

func main() {
	subject := "テスト"
	contents := "テストですよ"

	SendMail(subject, contents)
}

// メールの中身を作成して、メールを送信する
func SendMail(subject, contents string) {

	Env_load()

	// .envファイルに格納したAPI KEYを取得
	apiKey := os.Getenv("SENDGRID_API_KEY")
	// ホスト
	host := "https://api.sendgrid.com"
	// エンドポイント
	endpoint := "/v3/mail/send"

	// API KEYとエンドポイント、ホストからrestパッケージのRequestを生成
	request := sendgrid.GetRequest(apiKey, endpoint, host)
	// requestのMethodをPostに
	request.Method = "POST"

	// メールの内容をJSONで作成する
	mail := Mail{
		Subject: subject,
		Personalizations: []Personalizations{
			{To: []MailUser{{
				Email: os.Getenv("RECEIVER_USER_ADDRESS1"),
				Name:  os.Getenv("RECEIVER_USER_NAME1"),
			},
				{
					Email: os.Getenv("RECEIVER_USER_ADDRESS2"),
					Name:  os.Getenv("RECEIVER_USER_NAME2"),
				},
			}}},
		From: MailUser{
			Email: os.Getenv("SENDER_USER_ADDRESS"),
			Name:  os.Getenv("SENDER_USER_NAME"),
		},
		Content: []Contents{{
			Type:  "text/plain",
			Value: contents,
		}},
	}

	// GoのコードをJSON化
	data, err := json.Marshal(mail)

	log.Println(string(data))

	if err != nil {
		log.Println(err)
	}

	// JSON化したmailの内容をrequest.Bodyに代入
	request.Body = data

	// sendGridのAPIにリクエストをセット
	// 戻り値でresponseが返ってくる
	response, err := sendgrid.API(request)
	if err != nil {
		log.Println(err)
	} else {
		fmt.Println(response.StatusCode)
		fmt.Println(response.Body)
		fmt.Println(response.Headers)
	}
}

```


.envファイル（各項目は適当に置き換える）

```
SENDGRID_API_KEY="API KEYを書く"
SENDER_USER_NAME="送信者のユーザーネーム"
SENDER_USER_ADDRESS="送信者のメールアドレス"
RECEIVER_USER_NAME1="受信者1のユーザーネーム"
RECEIVER_USER_ADDRESS1="受信者1のメールアドレス"
RECEIVER_USER_NAME2="受信者2のユーザーネーム"
RECEIVER_USER_ADDRESS2="受信者2のメールアドレス"
```

今回は、2人に対してメールを送信している。

ちなみに、コード中にAPI KEYやメールアドレス、ユーザーネームを格納するのはセキュリティ上、あまりよろしくないと思ったので、[godotenv](https://github.com/joho/godotenv "godotenv") を使用して.envファイルから上記のようなデータを読み込んでいる。

コードにベタ打ちしたい場合は、各 `os.Getenv` の部分を生の文字列に置き換えれば良い。


基本的には、以下のコードを参考にさせていただいているが、JSONの各プロパティを動的に変更したいので、JSONをrow string literalではなく、構造体にしてからJSONへ変換している。
https://github.com/sendgrid/sendgrid-go/blob/master/examples/mail/mail.go 


JSONの各プロパティの説明については以下に記載されている。
https://sendgrid.com/docs/API_Reference/Web_API_v3/Mail/index.html

## 参考にさせていただいたサイト
 [SendGridとは](https://sendgrid.kke.co.jp/about "SendGridとは")
 [BREAKING CHANGESとは](https://github.com/sendgrid/sendgrid-go/issues/81 "BREAKING CHANGES")
 [SendGrid API KEY](https://sendgrid.kke.co.jp/docs/User_Manual_JP/Settings/api_keys.html "SendGrid API KEY")
 [godotenv](https://github.com/joho/godotenv "godotenv")
[SendGriddサンプル](https://github.com/sendgrid/sendgrid-go/blob/master/examples/mail/mail.go  "SendGridサンプル")
[SendGridリファレンス](https://sendgrid.com/docs/API_Reference/Web_API_v3/Mail/index.html  "SendGridリファレンス")


## 関連記事
 [GCE上のGoのコードからSendGridを使用してメールを送信](http://qiita.com/Sekky0905/items/ce9588b29ac9fca507c2 "GCE上のGoのコードからSendGridを使用してメールを送信")

※ ブログでも同一記事を投稿している<br>
[Go言語でSendGrid Web API v3を使って、メールを送信する](http://www.sekky0905.com/entry/2017/05/19/Go%E8%A8%80%E8%AA%9E%E3%81%A7SendGrid_Web_API_v3%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E3%80%81%E3%83%A1%E3%83%BC%E3%83%AB%E3%82%92%E9%80%81%E4%BF%A1%E3%81%99%E3%82%8B "Go言語でSendGrid Web API v3を使って、メールを送信する")



