# Natural Language API Client LibrariesをGoで実装してみる

## CLOUD NATURAL LANGUAGE API

### CLOUD NATURAL LANGUAGE APIとは
> Google Cloud Natural Language API は、使いやすい REST API でパワフルな機械学習モデルを適用し、テキストの構造や意味を認識します。

機械学習モデルを使用して、テキストを分析してくれる

引用元  [CLOUD NATURAL LANGUAGE API](https://cloud.google.com/natural-language/?hl=ja)

### CLOUD NATURAL LANGUAGE APIの機能
CLOUD NATURAL LANGUAGE APIの機能は主に3つある。
1. 感情分析(英語のみ)
2. エンティティ分析
3. 構文解析

詳細は、以下の公式ドキュメント(日本語)に記述されている。
 [Natural Language API の基本](https://cloud.google.com/natural-language/docs/basics?hl=ja)

### 今回やること
今回は、上述の3つの機能のうち、「構文解析」をGoのクライアントライブラリから使用してみる。

### 構文解析で何ができるのか
> 言語情報を抽出し、指定されたテキストを一連の文とトークン（通常は単語の境界）に分解して、それらのトークンをさらに分析できるようにします。

引用元: [Natural Language API の基本](https://cloud.google.com/natural-language/docs/basics?hl=ja)

### 構文解析の大まかな流れ
1. テキストがトークン（通常は単語の境界）に変換される。(トークン化)
2. 品詞（形態論的情報を含む）と基本形の関連する部分が特定される。
3. トークンが評価されて、依存関係ツリーに配置される。

以下より抜粋一部変更。
 [Natural Language API の基本](https://cloud.google.com/natural-language/docs/basics?hl=ja)


形態論と依存関係ツリーについては[ここ](https://cloud.google.com/natural-language/docs/morphology?hl=ja)を参照。

上記の流れが完了すると
> トークンの構文上の意味を特定したり、他のトークンとの関係や、トークンを含む文との関係を示したりできます。

引用元 https://cloud.google.com/natural-language/docs/basics?hl=ja



## 実装してみる

###  必要なライブラリを取得する

```
go get -u cloud.google.com/go/language/apiv1
```

参考
[Natural Language API Client Libraries](https://cloud.google.com/natural-language/docs/reference/libraries?hl=ja)


### 実装

``` go
package main

import (
	"cloud.google.com/go/language/apiv1"
	"encoding/json"
	"fmt"
	"golang.org/x/net/context"
	languagepb "google.golang.org/genproto/googleapis/cloud/language/v1"
	"log"
)

func main() {
	doAnalyze()
}

func doAnalyze() {
	// コンテキスト生成
	ctx := context.Background()

	// クライアント生成
	client, err := language.NewClient(ctx)

	if err != nil {
		log.Fatalf("Failed to create client: %v", err)
	}

	// 分析するテキストを宣言
	text := "Natural Language API Client LibrariesクライアントをGoで実装してみる。"

	// AnalyzeSyntaxRequest構造体を生成
	req := &languagepb.AnalyzeSyntaxRequest{
		Document: &languagepb.Document{
			// 指定なし 0
			// Plain text 1
			// HTML 2
			Type: 1,
			// 分析する内容
			Source: &languagepb.Document_Content{
				Content: text,
			},
		},
		EncodingType: languagepb.EncodingType_UTF8,
	}

	// 構文解析を行う
	resp, err := client.AnalyzeSyntax(ctx, req)
	if err != nil {
		log.Fatalf("Failed to AnalyzeSyntax: %v", err)
	}

	// jsonにする
	rawJson, err := json.Marshal(resp)

	if err != nil {
		log.Fatalf("Failed to Marshal json: %v", err)
	}

	jsonStr := string(rawJson)

	fmt.Println("構文解析の結果\n" + jsonStr)

}

```

参考
[Natural Language API Client Libraries](https://cloud.google.com/natural-language/docs/reference/libraries?hl=ja)

[package language](https://godoc.org/cloud.google.com/go/language/apiv1)


### 送信されてきたJSON

``` json
{
    "sentences": [
        {
            "text": {
                "content": "Natural Language API Client LibrariesクライアントをGoで実装してみる。"
            }
        }
    ],
    "tokens": [
        {
            "text": {
                "content": "Natural"
            },
            "part_of_speech": {
                "tag": 12,
                "proper": 1
            },
            "dependency_edge": {
                "head_token_index": 5,
                "label": 26
            },
            "lemma": "Natural"
        },
        {
            "text": {
                "content": "Language",
                "begin_offset": 8
            },
            "part_of_speech": {
                "tag": 6,
                "proper": 1
            },
            "dependency_edge": {
                "head_token_index": 2,
                "label": 26
            },
            "lemma": "Language"
        },
        {
            "text": {
                "content": "API",
                "begin_offset": 17
            },
            "part_of_speech": {
                "tag": 6,
                "proper": 1
            },
            "dependency_edge": {
                "head_token_index": 5,
                "label": 26
            },
            "lemma": "API"
        },
        {
            "text": {
                "content": "Client",
                "begin_offset": 21
            },
            "part_of_speech": {
                "tag": 6,
                "proper": 2
            },
            "dependency_edge": {
                "head_token_index": 4,
                "label": 26
            },
            "lemma": "Client"
        },
        {
            "text": {
                "content": "Libraries",
                "begin_offset": 28
            },
            "part_of_speech": {
                "tag": 6,
                "proper": 2
            },
            "dependency_edge": {
                "head_token_index": 5,
                "label": 26
            },
            "lemma": "Libraries"
        },
        {
            "text": {
                "content": "クライアント",
                "begin_offset": 37
            },
            "part_of_speech": {
                "tag": 6,
                "proper": 2
            },
            "dependency_edge": {
                "head_token_index": 9,
                "label": 18
            },
            "lemma": "クライアント"
        },
        {
            "text": {
                "content": "を",
                "begin_offset": 55
            },
            "part_of_speech": {
                "tag": 9,
                "case": 1,
                "proper": 2
            },
            "dependency_edge": {
                "head_token_index": 5,
                "label": 45
            },
            "lemma": "を"
        },
        {
            "text": {
                "content": "Go",
                "begin_offset": 58
            },
            "part_of_speech": {
                "tag": 12,
                "proper": 2
            },
            "dependency_edge": {
                "head_token_index": 9,
                "label": 64
            },
            "lemma": "Go"
        },
        {
            "text": {
                "content": "で",
                "begin_offset": 60
            },
            "part_of_speech": {
                "tag": 9,
                "case": 2,
                "proper": 2
            },
            "dependency_edge": {
                "head_token_index": 7,
                "label": 45
            },
            "lemma": "で"
        },
        {
            "text": {
                "content": "実装",
                "begin_offset": 63
            },
            "part_of_speech": {
                "tag": 6,
                "proper": 2
            },
            "dependency_edge": {
                "head_token_index": 9,
                "label": 54
            },
            "lemma": "実装"
        },
        {
            "text": {
                "content": "し",
                "begin_offset": 69
            },
            "part_of_speech": {
                "tag": 11,
                "form": 5,
                "proper": 2
            },
            "dependency_edge": {
                "head_token_index": 9,
                "label": 24
            },
            "lemma": "する"
        },
        {
            "text": {
                "content": "て",
                "begin_offset": 72
            },
            "part_of_speech": {
                "tag": 9,
                "proper": 2
            },
            "dependency_edge": {
                "head_token_index": 9,
                "label": 45
            },
            "lemma": "て"
        },
        {
            "text": {
                "content": "みる",
                "begin_offset": 75
            },
            "part_of_speech": {
                "tag": 11,
                "form": 4,
                "proper": 2
            },
            "dependency_edge": {
                "head_token_index": 9,
                "label": 66
            },
            "lemma": "みる"
        },
        {
            "text": {
                "content": "。",
                "begin_offset": 81
            },
            "part_of_speech": {
                "tag": 10,
                "proper": 2
            },
            "dependency_edge": {
                "head_token_index": 9,
                "label": 32
            },
            "lemma": "。"
        }
    ],
    "language": "ja"
}
```
返却されたJSONの各プロパティの意味の意味については、以下を参照。
 [Natural Language API の基本](https://cloud.google.com/natural-language/docs/basics?hl=ja)
 [形態論と依存関係ツリー](https://cloud.google.com/natural-language/docs/morphology?hl=ja)

## 参考にさせていただいたサイト
 [CLOUD NATURAL LANGUAGE API](https://cloud.google.com/natural-language/?hl=ja)

 [Natural Language API の基本](https://cloud.google.com/natural-language/docs/basics?hl=ja)

 [形態論と依存関係ツリー](https://cloud.google.com/natural-language/docs/morphology?hl=ja)

[Natural Language API Client Libraries](https://cloud.google.com/natural-language/docs/reference/libraries?hl=ja)

[package language](https://godoc.org/cloud.google.com/go/language/apiv1)

## 関連記事
[GoでBigQueryクライアントを実装してBigQueryからデータを取得する](http://qiita.com/Sekky0905/items/fd6ff9113d301aaa9e1d)

