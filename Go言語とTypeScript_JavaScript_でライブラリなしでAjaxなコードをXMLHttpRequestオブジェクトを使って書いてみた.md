# Go言語とTypeScript(JavaScript)でライブラリなしでAjaxなコードをXMLHttpRequestオブジェクトを使って書いてみた
ライブラリなしで、GoとjavaScript(実際に実装したのはTypeScriptですが)を使って本当に簡単なAjaxなアプリ?を書いた<br>

なお、コードの詳細な説明はコード中のコメントに記載していて、本文では大まかな流れの説明のみを記載している<br>

## プロジェクト構成
``` 
apprppt
      |_source
      |      |_index.js 
      |      |_index.ts
      |      |_package.json
      |      |_tsconfig.json
      |_view
      |    |_index.html
      |_main.go
``` 

## フロント側の設定ファイル
package.json

``` json
{
  "name": "ajax",
  "version": "1.0.0",
  "description": "ajax",
  "main": "./index.js",
  "scripts": {
    "start": "",
    "build": "tsc index.ts"
  },
  "author": "",
  "license": "MIT",
  "devDependencies": {
    "typescript": "^2.1.6"
  },
  "dependencies": {},
  "directories": {},
  "repository": {
    "type": "git",
    "url": "github.など"
  },
  "author": "",
  "license": "MIT",
  "bugs": {
    "url": "github.など"
  },
  "homepage": "github.など"
}
```

tsconfig.json

``` json
{
    "compilerOptions": {
        "module": "commonjs",
        "moduleResolution": "node",
        "target": "es5",
        "declaration": true,
        "sourceMap": true,
        "lib": [
            "dom",
            "es5"
        ],
        "noImplicitAny": true,
        "strictNullChecks": false,
        "noFallthroughCasesInSwitch": true,
        "noImplicitReturns": true,
        "noImplicitThis": true,
        "noUnusedLocals": true,
        "noUnusedParameters": true,
        "noImplicitUseStrict": false,
        "emitDecoratorMetadata": true,
        "experimentalDecorators": true,
        "forceConsistentCasingInFileNames": true,
        "traceResolution": false,
        "listFiles": false,
        "stripInternal": true,
        "skipDefaultLibCheck": true,
        "skipLibCheck": false,
        "pretty": false,
        "noEmitOnError": true
    },
    "include": [
        "./**/*.ts"
    ],
    "types": [
    ],
    "exclude": [
        "node_modules",
        "code/definition-file/usage/",
        "code/definition-file/augmentGlobal/",
        "code/definition-file/issue9831/",
        "code/**/*-invalid.ts",
        "code/**/*-invalid.d.ts",
        "code/**/invalid.ts",
        "code/**/invalid.d.ts",
        "code/**/*-ignore.ts",
        "code/**/*-ignore.d.ts",
        "code/**/ignore.ts",
        "code/**/ignore.d.ts"
    ]
}
```

## HTMLファイル

index.html

``` html
<!DOCTYPE html>
<html lang="ja">

<head>
    <meta charset="UTF-8">
    <title>Ajax</title>
</head>

<body>
<p id="click-here"> click here</p>
      <script type="text/javascript" src="../source/index.js"></script>
</body>

</html>
```

「click here」をクリックすると、「非同期」というポップアップが表示され、その後にJsonを反映した以下の文字が表示される<br>

``` html
名前 : Go
型　: Static
```

## TypeScriptファイル

index.ts

``` ts
const id = 'click-here';
// idで指定して、elemntを取得
const e = document.getElementById(id);
// イベントリスナー登録
// クリックされたら、コールバック関数発動
e.addEventListener('click', () => {
    // 生のjavaScriptでAjaxを行うためには、XMLHttpRequestを使用する

    // XMLHttpRequestインスタンスの生成
    const httpRequest = new XMLHttpRequest();

    // openメソッド : リクエストを初期化する
    // 第1引数で、GETかPOSTのリクエストメソッドを指定
    // 第2引数で、"リクエスト先のURL"
    // 第3引数で、同期か非同期かの選択（非同期の場合は、true）
    httpRequest.open("GET", "/json", true)

    // readyStateプロパティ : XMLHttpRequestオブジェクト(HTTPリクエスト)のサーバとの通信の状態
    // この値がサーバとの通信状態によって変化する
    // readyStateプロパティの値が変化するたびにonreadystatechangeイベントが発生する
    // このonreadystatechangeイベントが発生したら、readyStateプロパティをチェックすれば、現在の通信状態を知ることができる

    console.log(httpRequest.readyState);

    // XMLHttpRequestオブジェクト(HTTPリクエスト)のonreadystatechangeイベントが発生したら、=で与えた関数が発動する
    httpRequest.onreadystatechange = () => {

        // 通信が完了したならば
        if (httpRequest.readyState === 4 && httpRequest.status === 200) {
            // jsonをtextデータとして受け取る
            const jsonText = httpRequest.responseText

            // textデータとして受け取ったjsonをパース
            const json = JSON.parse(jsonText);

            // 通信が完了（受信）したら、行う処理を記載
            e.innerHTML = `
            <p>名前: ${json["Name"]}</p>
            <p>型　: ${json["LangType"]}</p>
            `
        }
    }
    // sendメソッド : openで作成したHTTPリクエストをサーバへ送信する
    // POSTの場合は、送信するデータを引数で与える
    // GETの場合は、nullを引数で与える
    httpRequest.send(null);

    // 非同期のテスト
    // 非同期で通信しているので、HTMLの<p>が変化するよりもこちらの方が先に呼び出される
    window.alert('非同期');

})
```

### index.tsでやっていること
``` <p id="click-here"> click here</p> ```に対して、イベントリスナーを登録して、「click here」をクリックすると ``` e.addEventListener('click', () => {} ``` で登録しているコールバック関数が呼び出される<br>


コールバック、非同期処理に関しては以下の記事がわかりやすい<br>
http://qiita.com/YoshikiNakamura/items/732ded26c85a7f771a27
http://qiita.com/kiyodori/items/da434d169755cbb20447

コールバック関数内で、XMLHttpRequestオブジェクトを生成してクライアントからサーバへリクエストを送信している<br>
> XMLHttpRequest は、クライアントとサーバーの間でデータを伝送するための機能をクライアント側で提供する API です。<br>
https://developer.mozilla.org/ja/docs/Web/API/XMLHttpRequest (引用元)<br>

今回はXMLHttpRequestで非同期でクライアントからサーバへリクエストを送信しているので、クリックされると、後に記載されている ``` window.alert('非同期'); ``` が先に呼ばれて、```  <p id="click-here"> click here</p> ```の要素がJsonの値を反映したものに変化するより先に、windowのalertがポップアップされる<br>

その他のXMLHttpRequestオブジェクトを使用したクライアントからサーバへのリクエストの送信処理の詳しいことは、コード中にコメントとして記載している<br>


## Goファイル

main.go

``` go
package main

import (
    "fmt"
    "net/http"
    "html/template"
    "encoding/json"
)

func main() {
    fmt.Println("The Server runs with http://localhost:8080/")
    http.Handle("/source/", http.StripPrefix("/source/", http.FileServer(http.Dir("source/"))))
    http.HandleFunc("/", Handler)
    http.HandleFunc("/json", JsonHandler)
    http.ListenAndServe(":8080", nil)
}
// 構造体
type Language struct {
    Name     string
    LangType string
}

func Handler(w http.ResponseWriter, r *http.Request) {

    tmpl := template.Must(template.ParseFiles("./view/index.html"))
    tmpl.Execute(w, nil)
}

func JsonHandler(w http.ResponseWriter, r *http.Request) {

    // 構造体をインスタンス化
    l := Language{Name : "Go", LangType: "Static"}

    // Header() : WriteHeaderによって送信されるheaderのmapを返す
    // func (h Header) Set(key, value string)
    // headerエンティティのkeyに対してvalueを関連付ける
    w.Header().Set("Content-Type", "application/json; charset=utf-8")

    // WriteHeader(int) : ステータスコードと共にHTTPのレスポンスヘッダを送信する
    // http.StatusOKは200が設定されている
    w.WriteHeader(http.StatusOK)

    // Encoderは、出力ストリームにJSONオブジェクトを書き込む
    // NewEncoder(w io.Writer) : wに書き込みを行い、新しいEncoderを返す
    // Encode() : 引数のJSONエンコーディングをWriterに書き込む
    json.NewEncoder(w).Encode(l)
}
```

### main.goでやっていること
XMLHttpRequestオブジェクトでリクエストが送信され、"/json"のURLに登録されているハンドラが発動する<br>
構造体のフィールドを元に、JSONにしレスポンスしている<br>


## 参考にさせていただいたサイト
https://developer.mozilla.org/ja/docs/Web/API/XMLHttpRequest
http://qiita.com/katsunory/items/9bf9ee49ee5c08bf2b3d
http://www.ajaxtower.jp/ini/
http://www.ajaxtower.jp/ini/http/index2.html
http://www.ajaxtower.jp/ini/http/index4.html

※ ブログでも同一記事を投稿している
http://www.sekky0905.com/entry/2017/02/25/Go%E8%A8%80%E8%AA%9E%E3%81%A8TypeScript%28JavaScript%29%E3%81%A7%E3%83%A9%E3%82%A4%E3%83%96%E3%83%A9%E3%83%AA%E3%81%AA%E3%81%97%E3%81%A7Ajax%E3%81%AA%E3%82%B3%E3%83%BC%E3%83%89%E3%82%92%E6%9B%B8%E3%81%84
