# GCE上のGoのコードからSendGridを使用してメールを送信
GCE上のGoのコードからSendGridを使用してメールを送信する。
SendGridによるGCPからのメール送信は既に以下の公式ドキュメントや記事で丁寧に紹介されている。
https://cloud.google.com/compute/docs/tutorials/sending-mail/using-sendgrid
http://www.apps-gcp.com/sendgrid-gce/

ただ、公式ドキュメント等で紹介されているのは、Node.jsやJavaでのサンプルである。
僕はGoが好きなので、今回はGCEのインスタンス上のGoを使用してメールを送信する方法を記す。

## なぜ、わざわざ3rdパーティー製のプロバイダを利用しなければならないのか
GCEでは標準メールポートを使用することができないため。
>Google Compute Engine では、ポート 25、465、587 での送信接続は許可されません。これらの送信 SMTP ポートは、不正使用に利用されることが多いため、デフォルトでブロックされています。

引用元　https://cloud.google.com/compute/docs/tutorials/sending-mail/

## 大まかな流れ
今回は、大きく以下のような手順で行う
1. GCEでインスタンス(仮想マシン)作成
2. SendGridでの手続き
3. GCEでインスタンスでSendGridの設定
4. GCEでインスタンス上にGoの環境を構築
5. SendGridのGoライブラリを使って、Goのコードを作成
6. Goのコードでメールを送信

## 1. GCEでインスタンス(仮想マシン)作成
### GCEとは
> Google のデータセンターとファイバー ネットワークで運用される仮想マシンを提供します

引用元 : https://cloud.google.com/compute/?hl=ja

### インスタンスとは
インスタンスとは、Google のインフラがホストしている仮想マシン(Vertial Machine)のこと
以下の公式ドキュメントが日本語で詳しく記述されている
https://cloud.google.com/compute/docs/instances/?hl=ja

仮想マシンについては以下のサイトが分かりやすい
https://boxil.jp/others/a1300

### インスタンス仮想マシン(Vertial Machine)を作成する
以下の公式ドキュメントの通りにやっていけば、VMインスタンスを生成することができる
https://cloud.google.com/compute/docs/instances/create-start-instance?hl=ja

## 2. SendGridでの手続き
### SendGridとは
メール配信サービス。
APIを通じて、SendGridのクラウド上のインフラからメールを送信することができる。
https://sendgrid.kke.co.jp/about/

### SendGridに登録
登録に関しては既に詳しい記事が存在するので、以下の記事を参照
http://www.apps-gcp.com/sendgrid-gce/


## 3. GCEでインスタンスでSendGridの設定
ここから、先ほど作成したGCE上のインスタンスに対して、諸々の設定を行う。
既に日本語で丁寧な説明が以下の公式ドキュメントでなされているので、以下を参照して進めていく。
https://cloud.google.com/compute/docs/tutorials/sending-mail/using-sendgrid


## 4. GCEでインスタンス上にGoの環境を構築
1.Goをインストール、GOPATHの設定
Goを[インストール(Cent OSの場合)](http://qiita.com/ikenyal/items/eecc65b703eba8a52e88  "インストール(Cent OSの場合)")

Cent OSの場合、wgetが入っていないことがあるので、その場合は以下のコマンドを実施

```
yum install wget
```

2.必要なpackageをgo get 
今回は、以下

```
go get -u github.com/joho/godotenv
go get -u github.com/sendgrid/sendgrid-go
```

以上で、GCEでインスタンス上でGoを使用できるようになった

## 5.SendGridのGoライブラリを使って、Goのコードを作成
[この記事](http://qiita.com/Sekky0905/items/7f7285d0a2b3e5f21e57  "この記事")で本記事の以下のGoのコードの詳しい説明をしている。

goのコード


``` go:sendgrid.go
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

.envファイル

```:.env
SENDGRID_API_KEY="API KEYを書く"
SENDER_USER_NAME="送信者のユーザーネーム"
SENDER_USER_ADDRESS="送信者のメールアドレス"
RECEIVER_USER_NAME1="受信者1のユーザーネーム"
RECEIVER_USER_ADDRESS1="受信者1のメールアドレス"
RECEIVER_USER_NAME2="受信者2のユーザーネーム"
RECEIVER_USER_ADDRESS2="受信者2のメールアドレス"
```

やっている大まかな流れは、
1. 取得したAPI KEYとエンドポイント、ホストからrestパッケージのRequestを生成
2. リクエストのメソッドを決定
3. リクエストのボディで、送信するメールのメタデータをjsonで作成する
4. 作成したリクエストを送信

とまあ、一般的なHTTP通信に則った手順でメールを送信するためにSendGrid APIにリクエストを送信することができる。

## 6. Goのコードでメールを送信
1. GCEのインスタンスのコマンドラインを起動
2. `go build sendgrid.go`
3. `go run sendgrid.go`

上記を行うと、 `sendgrid.go` で指定した送信先のユーザーのメールアドレスにメールが送信されている


## 参考にさせていただいたサイト
[SendGrid でのメールの送信/公式ドキュメント](https://cloud.google.com/compute/docs/tutorials/sending-mail/using-sendgrid  "SendGrid でのメールの送信/公式ドキュメント")
[SendGridを使ってGCEからメールを確実に送る方法](http://www.apps-gcp.com/sendgrid-gce/ "SendGridを使ってGCEからメールを確実に送る方法")
[インスタンスからのメールの送信](https://cloud.google.com/compute/docs/tutorials/sending-mail/ "インスタンスからのメールの送信")
[COMPUTE ENGINE](https://cloud.google.com/compute/?hl=ja "COMPUTE ENGINE")
[仮想マシン インスタンス](https://cloud.google.com/compute/docs/instances/?hl=ja "仮想マシン インスタンス")
[5分で解る「仮想マシン」!!今更人に聞けないキホンを説明](https://boxil.jp/others/a1300 "5分で解る「仮想マシン」!!今更人に聞けないキホンを説明")
[SendGridとは](https://sendgrid.kke.co.jp/about/ "SendGridとは")
[CentOSにGo言語をインストール](http://qiita.com/ikenyal/items/eecc65b703eba8a52e88  "CentOSにGo言語をインストール")
 [godotenv](https://github.com/joho/godotenv "godotenv")

## 関連記事
 [Go言語でSendGrid Web API v3を使って、メールを送信する](http://qiita.com/Sekky0905/items/7f7285d0a2b3e5f21e57 "Go言語でSendGrid Web API v3を使って、メールを送信する")

※ ブログでも同一記事を投稿している<br>
[GCE上のGoのコードからSendGridを使用してメールを送信](http://www.sekky0905.com/entry/2017/05/19/GCE%E4%B8%8A%E3%81%AEGo%E3%81%AE%E3%82%B3%E3%83%BC%E3%83%89%E3%81%8B%E3%82%89SendGrid%E3%82%92%E4%BD%BF%E7%94%A8%E3%81%97%E3%81%A6%E3%83%A1%E3%83%BC%E3%83%AB%E3%82%92%E9%80%81%E4%BF%A1 "GCE上のGoのコードからSendGridを使用してメールを送信")


