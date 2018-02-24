# HTTPとPOSTとGET

#HTTPとは

## HTTPの概要
>HTTPとは、WebサーバとWebクライアントの間でデータの送受信を行うために用いられるプロトコル
(引用元)http://e-words.jp/w/HTTP.html


* プロトコルとは、難しく言うと通信規約、簡単に言えば約束事や取り決めのこと
* つまり、Webサーバとクライアントは、HTTPというプロトコル(決まった方法)でやりとりしましょうねということ


## HTTP通信の中身
* HTTPでサーバとクライアントがやりとりするメッセージ（HTTPメッセージ）は基本的に以下のような形になっている
 
``` 
メッセージヘッダ
空行(CR+LF)
メッセージボディ
``` 

* リクエストメッセージ（クライアント=>サーバへのメッセージ）は具体的には以下のようなものである

```
GET /index.html HTTP/1.1 
Host: localhost:8080
Connection: keep-alive
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.98 Safari/537.36
Accept: */*
Referer: http://localhost:8080/
Accept-Encoding: gzip, deflate, sdch, br
Accept-Language: ja,en-US;q=0.8,en;q=0.6
```

※今回は、リクエストメソッドがGETのため、リクエストボディは存在しない（存在する場合については後述する）

### リクエストヘッダ
* 2行目~ 空行までの部分
* リクエストについての情報や属性が書かれている

###　リクエストライン
* 1行目の以下の部分


```
GET /index.js HTTP/1.1
```

このステータスラインの個々の部品を左から

 * GET = リクエストメソッド
 * /index.html = リクエストURI
 * HTTP/1.1 = HTTPのバージョン
という<br>

#### リクエストメソッド
* クライアントからサーバに対するリクエストの方法（基本的にGETかPOST）


####  リクエストURI 
* クライアントからサーバに対するリクエスト対象のリソース（サーバ内に保管されているリソース）


####   HTTPのバージョン 
* この通信で使うHTTPのバージョン

* 簡単に言うと、/index.htmlというリソースを、HTTPの1.1バージョンの通信で、GETというリクエストの方法で、クライアントからサーバに要求している。

# GETとPOST
HTTPのメソッドは、その用途によってGETとPOSTで使い分ける必要がある。(他にもPUTとかDELETEとかあるけど、本記事ではGETとPOSTだけ扱う)

## GET
* HTTP通信で、サーバから情報を取得してくる時に使用する
* 他人に見られたくない情報は、GETでは送らない（後述する）
* 送信できるデータ量に制限がある
* ブックマークに保存する場合
* テキストデータのみ送信できる（バイナリデータは送信できない）

### GETでのサーバへのデータの送信方法
* リクエスト時にサーバへ送信するデータはリクエストURLの後に付け加えられる


``` 
http://localhost:8080/index.html?hoge=hoge
```

* ?より後の文字をクエリストリングといい、送信するデータを表す（クエリストリングについては後述）
* このように送信するデータがアドレスバーに表示されてしまうため、他人に見られる可能性があるので、他人に見られたくない情報は、GETでは送らない

### クエリストリング
> クエリ文字列とは、WebブラウザなどがWebサーバに送信するデータをURLの末尾に特定の形式で表記したもの。
Webアプリケーションなどでクライアントからサーバにパラメータを渡すのに使われる表記法で、URLの末尾に「?」マークを付け、続けて「名前=値」の形式で記述する。値が複数あるときは「&」で区切り、<br>
例えば以下のように記述する

```
http://example.com/foo?name1=value1&name2=value2 
```
(引用元)
http://e-words.jp/w/%E3%82%AF%E3%82%A8%E3%83%AA%E6%96%87%E5%AD%97%E5%88%97.html


下記の?hoge=1&foo=2がクエリストリングに該当する

``` 
http://localhost:8080/index.html?hoge=1&foo=2
```

* 「名前=値」の形で表される<br>
この場合は以下の2つのデータを送信していることがわかる<br>

```
「hogeが名前、1が値」、「fooが名前、2が値」
```

##　パーセントエンコーディング（URLエンコーディング）
* 日本語などの文字は、そのままではURLで送信することができない
* そのような文字をURLに付与して送信するには、パーセントエンコーディング（URLエンコーディング）という技術を使用する。
* URLで扱えない文字を「%xx」（xx:16進数）で表現する
* XXの部分は、使用する文字コードによって異なる<br>

このパーセントエンコーディングはHTTP通信においては、以下の箇所で利用される
1. URLのパス
2. クエリストリング（GETパラメータ部分）
3. リクエストボディ（POSTパラメータ）



## POST
* HTTP通信で、サーバへ情報を登録する時に使用する（データベースへの格納など）
* データ量が多い場合（GETでのデータ送信量制限を超えてしまう場合）
* バイナリデータを送信したい場合
* 他の人に見られたくない情報を送る場合（パスワードなど）

### POSTでのサーバへのデータの送信方法
* POSTでサーバにデータを送信する際は、リクエストボディにデータが記述される
* POSTでのサーバからクライアントへのリクエスト例


```
POST /hoge/ HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Content-Length: 22
Cache-Control: max-age=0
Origin: http://localhost:8080
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.98 Safari/537.36
Content-Type: application/x-www-form-urlencoded
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Referer: http://localhost:8080/hoge/
Accept-Encoding: gzip, deflate, br
Accept-Language: ja,en-US;q=0.8,en;q=0.6

name=hoge&comment=hoge
```
* 上記のように、リクエストヘッダの後に一行、空行が入り、その後POSTで送信したクエリストリングが、リクエストボディとしてクライアントからサーバへと送信されてくる
* リクエストボディの長さは、「Content-Length:」という項目で表される


## Webサーバ（HTTPサーバ）とは
* クライアントからのリクエストに対して静的なリソースを返す
* 動的なリソースを返したい場合は、アプリケーションサーバ

# 参考にさせていただいたサイト

https://www.ietf.org/rfc/rfc2616.txt(rfc2616)
https://triple-underscore.github.io/RFC2616-ja.html
https://www.seohacks.net/blog/seo-tech/1147/
https://www.ietf.org/rfc/rfc3986.txt
http://www.atmarkit.co.jp/ait/articles/0801/18/news124.html

# 参考文献
* 上野　宣（2013/5/25）『HTTPの教科書 』翔泳社
* 小森 裕介　(2010/4/10)『「プロになるためのWeb技術入門」 ――なぜ、あなたはWebシステムを開発できないのか』技術評論社


※ ブログでも同一記事を投稿している
http://www.sekky0905.com/entry/2016/12/08/%E3%80%90HTTP%E3%80%91HTTP%E3%81%A8GET%E3%80%81POST










