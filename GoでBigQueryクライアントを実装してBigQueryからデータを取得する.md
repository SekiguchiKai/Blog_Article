# GoでBigQueryクライアントを実装してBigQueryからデータを取得する
会社のある偉大な先輩は、以前こう言った。
「BigQueryは全人類が使えるようになるべきだ」と。

僕は、最近BigQueryの学習を始めた。
BigQueryの学習の一環として、BigQueryクライアントを実装してみた。

## やること
GoのコードからBigQueryクライアントを実装してSQLを走らせ、データをローカルに取得して、コマンドラインに表示させる。

## クライアントライブラリって何?
好みの言語から、Google Cloud APIsをHTTPに直接リクエストしたり、RPC callsしたりして使うためのライブラリ。
[公式ドキュメント](https://cloud.google.com/apis/docs/client-libraries-explained?hl=ja "公式ドキュメント")


## 準備

### クライアントライブラリのインストール
BigQueryのクライアントライブラリを以下のコマンドでインストールする。

```
go get cloud.google.com/go/bigquery
```

### 認証
BigQueryのAPIを使用するためには、アカウント認証を行う必要があるので、それを行う。
以下のサイトが参考になる。
[公式ドキュメント](https://cloud.google.com/bigquery/docs/reference/libraries?hl=ja#client-libraries-install-go "公式ドキュメント")

[FluentdでGoogle BigQueryにログを挿入してクエリを実行する](http://qiita.com/harukasan/items/0e0da49c65839e9517c4 "FluentdでGoogle BigQueryにログを挿入してクエリを実行する")


[Google Cloud Platformをローカルから利用するための準備](http://qiita.com/kosuke_nishaya/items/3d9a95f559d0c22d8134 "Google Cloud Platformをローカルから利用するための準備")


## BigQueryクライアントを使う

### 手順
 BigQueryクライアントをGoで実装するためには、大まかに以下のような手順を踏む。

1. 空のcontextを生成
2. contextとprojectIDを元にBigQuery用のclientを生成
3. Queryを生成
4. Queryを読み込んで、resultを取得するためのIteratorを生成
5. Iteratorを回して順次結果を取得

詳細は、コード中にコメントとして記述している

なお、以下の公式ドキュメントを参考にしている


[package bigqueryのGoDoc](https://godoc.org/cloud.google.com/go/bigquery "package bigqueryのGoDoc")

[BigQuery クライアント ライブラリの公式ドキュメント](https://cloud.google.com/bigquery/docs/reference/libraries?hl=ja#client-libraries-usage-go
 "BigQuery クライアント ライブラリの公式ドキュメント")

コード

``` go
package main

import (
	"cloud.google.com/go/bigquery"
	"context"
	"fmt"
	"google.golang.org/api/iterator"
	"log"
)

func main() {
	fetchBigQueryData()
}

// BigQueryで実行するためのSQL
const QUERY = `
SQL
`

func fetchBigQueryData() {
	// 空のcontextを生成
	ctx := context.Background()

	// プロジェクトのID
	projectID := "プロジェクトのID"

	// contextとprojectIDを元にBigQuery用のclientを生成
	client, err := bigquery.NewClient(ctx, projectID)

	if err != nil {
		log.Printf("Failed to create client:%v", err)
	}

	// 引数で渡した文字列を元にQueryを生成
	q := client.Query(QUERY)

	// 実行のためのqueryをサービスに送信してIteratorを通じて結果を返す
	// itはIterator
	it, err := q.Read(ctx)

	if err != nil {
		log.Println("Failed to Read Query:%v", err)
	}

	for {
		// BigQueryの結果から、中身を格納するためのBigQuery.Valueのsliceを宣言
		// BigQuery.Valueはinterface{}型
		var values []bigquery.Value

		// 引数に与えたvaluesにnextを格納する
		// Iteratorを返す
		// これ以上結果が存在しない場合には、iterator.Doneを返す
		// iterator.Doneが返ってきたら、forを抜ける
		err := it.Next(&values)
		if err == iterator.Done {
			break
		}

		if err != nil {
			log.Println("Failed to Iterate Query:%v", err)
		}

		fmt.Println(values)
	}

}
```


### 実際に簡単なQueryを投げるように実装してみた
今回は、BigQueryのコンソールにデフォルトで表示されるPublic Datasetsの中から
bigquery-public-data:usa_namesのusa_1910_2013のデータを使用した。

このテーブルに対して、'Mary’という名前を持つデータを取得して、コマンドラインに表示する。

実装の流れは先ほどと全く変わらない。

今回は、BigQueryクライアントの実装を目的としているため、SQLは以下のようにかなり簡素なものになっている。

SQL

``` sql
SELECT
  *
FROM
  [bigquery-public-data:usa_names.usa_1910_2013]
WHERE
  name = 'Mary'
LIMIT
  100
```


実装

``` go
package main

import (
	"cloud.google.com/go/bigquery"
	"context"
	"fmt"
	"google.golang.org/api/iterator"
	"log"
)

func main() {
	fetchBigQueryData()
}

// BigQueryで実行するためのSQL
const QUERY = `
SELECT  *
FROM [bigquery-public-data:usa_names.usa_1910_2013]
WHERE name = 'Mary'
LIMIT 10
`

func fetchBigQueryData() {
	// 空のcontextを生成
	ctx := context.Background()

	// プロジェクトのID
	projectID := "プロジェクトのID"

	// contextとprojectIDを元にBigQuery用のclientを生成
	client, err := bigquery.NewClient(ctx, projectID)

	if err != nil {
		log.Printf("Failed to create client:%v", err)
	}

	// 引数で渡した文字列を元にQueryを生成
	q := client.Query(QUERY)

	// 実行のためのqueryをサービスに送信してIteratorを通じて結果を返す
	// itはIterator
	it, err := q.Read(ctx)

	if err != nil {
		log.Println("Failed to Read Query:%v", err)
	}

	for {
		// BigQueryの結果から、中身を格納するためのBigQuery.Valueのsliceを宣言
		// BigQuery.Valueはinterface{}型
		var values []bigquery.Value

		// 引数に与えたvaluesにnextを格納する
		// Iteratorを返す
		// これ以上結果が存在しない場合には、iterator.Doneを返す
		// iterator.Doneが返ってきたら、forを抜ける
		err := it.Next(&values)
		if err == iterator.Done {
			break
		}

		if err != nil {
			log.Println("Failed to Iterate Query:%v", err)
		}

		fmt.Println(values)
	}

}

```

実行結果

``` go
[AK F 1912 Mary 9]
[AK F 1916 Mary 18]
[AK F 1918 Mary 27]
[AK F 1919 Mary 22]
[AK F 1923 Mary 26]
[AK F 1925 Mary 24]
[AK F 1926 Mary 39]
[AK F 1927 Mary 30]
[AK F 1930 Mary 35]
[AK F 1931 Mary 41]
```



## 参考にさせていただいたサイト
[package bigqueryのGoDoc](https://godoc.org/cloud.google.com/go/bigquery "package bigqueryのGoDoc")

[BigQuery クライアント ライブラリの公式ドキュメント](https://cloud.google.com/bigquery/docs/reference/libraries?hl=ja#client-libraries-usage-go
 "BigQuery クライアント ライブラリの公式ドキュメント")

[Google BigQuery API へのリクエストを認証する](https://cloud.google.com/bigquery/authentication?hl=ja#bigquery-authentication-go "Google BigQuery API へのリクエストを認証する")

[FluentdでGoogle BigQueryにログを挿入してクエリを実行する](http://qiita.com/harukasan/items/0e0da49c65839e9517c4 "FluentdでGoogle BigQueryにログを挿入してクエリを実行する")

※ ブログでも同一記事を投稿している<br>
[GoでBigQueryクライアントを実装してBigQueryからデータを取得する](http://www.sekky0905.com/entry/2017/05/15/Go%E3%81%A7BigQuery%E3%82%AF%E3%83%A9%E3%82%A4%E3%82%A2%E3%83%B3%E3%83%88%E3%82%92%E5%AE%9F%E8%A3%85%E3%81%97%E3%81%A6BigQuery%E3%81%8B%E3%82%89%E3%83%87%E3%83%BC%E3%82%BF%E3%82%92%E5%8F%96%E5%BE%97%E3%81%99 "GoでBigQueryクライアントを実装してBigQueryからデータを取得する")



