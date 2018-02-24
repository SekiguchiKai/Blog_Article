# ExpressなしでNode.jsで簡単なWebサーバを作ってみる、HTTPを学ぶ

#Webサーバとは
http://qiita.com/Sekky0905/items/29fb94743f11a212d395

#今回の目的
* 何のフレームワークもなしでwebサーバを書くには何が必要か知るため<br>


#Node.jsで簡単なWebサーバを作ってみる

###今回実現する機能
1. 指定されたポート(8080)でコネクションの受け入れを開始する
2. クライアントからHTTP通信でリクエストを受け取る
3. リクエストヘッダで送られてきたURLのパスを解析する
4. 解析したURLのパターンによって、そのパターンにバンドルした処理を呼びだす(リクエストハンドラ）<br>
=>本来Webアプリを作成する場合は、この処理を高度化し、別のファイルやモジュールに切り出す
5. 呼び出された処理を行う<br>
=>今回は単純にHTMLをレスポンスとして返す


###ファイルの構成

```
App
  ├ webServer.js
  └ template
        ├ index.html
        └ result.html
```

#ソースコード

webServer.js

```js
// クライアントからのレスポンスを受け取り、適切なファイルに処理を依頼する

// 必要なファイルを読み込み
var http = require('http');
var url = require('url');
var fs = require('fs');
var server = http.createServer();


// http.createServerがrequestされたら、(イベントハンドラ)
server.on('request', function (req, res) {
    // Responseオブジェクトを作成し、その中に必要な処理を書いていき、条件によって対応させる
    var Response = {
        "renderHTML": function () {
            var template = fs.readFile('./template/index.html', 'utf-8', function (err, data) {
                // HTTPレスポンスヘッダを出力する
                res.writeHead(200, {
                    'content-Type': 'text/html'
                });

                // HTTPレスポンスボディを出力する
                res.write(data);
                res.end("HTML file has already sent to browser");

            });

        },
        "resultGenerator": function () {
            var template = fs.readFile('./template/result.html', 'utf-8', function (err, data) {
                // HTTPレスポンスヘッダを出力する
                res.writeHead(200, {
                    'content-Type': 'text/html'
                });

                // HTTPレスポンスボディを出力する
                res.write(data);
                res.end("HTML file has already sent to browser");
        })
    }
};
    // urlのpathをuriに代入
    var uri = url.parse(req.url).pathname;


    // URIで行う処理を分岐させる
    if (uri === "/") {
        // URLが「http://localhost:8080/」の場合、"renderHTML"の処理を行う
        Response["renderHTML"](); 
        return;
    } else if (uri === "/result") {
        // URLが「http://localhost:8080/result」の場合、"resultGenerator"の処理を行う
        Response["resultGenerator"]();
        return;
    };
});

// 指定されたポート(8080)でコネクションの受け入れを開始する
server.listen(8080)
console.log('Server running at http://localhost:8080/');
```

index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>シンプルなwebサーバ</title>
</head>
<body>
    <p>Webサーバできたよ!!</p>
    <a href="http://localhost:8080/result">次のページ</a>
</body>
</html>
```

result.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>result!</title>
</head>
<body>
    <p>result.html見られたよ</p>
</body>
</html>
```


#HTTP周り
1. まず、クライアントから```http://localhost:8080/```にリクエストが送られてくる（GETメソッド）

2. 各URL毎にバンドルされた処理が行われる

3. 処理の一部として、クライアントにレスポンスを返す。
=>```res.writeHead```でレスポンスヘッダを
=>```res.write```でレスポンスボディ（HTML）を

# 参考にさせてもらったサイト
###HTTP周り
http://viral-community.com/other-it/http-1873/

###Node.js周り
http://uraway.hatenablog.com/entry/2015/11/03/node.js%E3%81%A7web%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC%E3%82%92%E7%AB%8B%E3%81%A6%E3%82%8B%E3%81%BE%E3%81%A7%E3%82%92%E7%90%86%E8%A7%A3%E3%81%99%E3%82%8B

http://engineer.recruit-lifestyle.co.jp/techblog/2015-06-22-node1/

http://kaworu.jpn.org/javascript/node.js%E3%81%AB%E3%82%88%E3%82%8BHTTP%E3%82%B5%E3%83%BC%E3%83%90%E3%81%AE%E4%BD%9C%E3%82%8A%E6%96%B9

# 参考文献
山本 陽平(2010/4/8)『Webを支える技術 -HTTP、URI、HTML、そしてREST (WEB+DB PRESS plus)』技術評論社



 
