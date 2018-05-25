# GCSライフサイクル機能を使ってオブジェクトをファイルを定期削除する

GCS (Google Cloud Storage)上のオブジェクトを自動削除するために、gsutilを利用してライフサイクルを設定する

## 何ができるの
便利なオンラインストレージであるGCS(Google Cloud Storage)。
しかし、データをずっと溜めっぱなしだと課金金額がかさんでいく。

「いちいち手動で削除するのも面倒だし~」という場合に、簡単なjsonを作成し、gsutilというツールを使用してコマンドを打つだけで、簡単に自動削除設定することができる。

これを設定すれば、例えば「1年以上前のデータは保存する必要ないから自動で削除したいなー」という要望を簡単に満たすことができるのだ。

## gsutilツールとは
> コマンドラインから Cloud Storage にアクセスできる Python アプリケーションです

引用元
https://cloud.google.com/storage/docs/gsutil

## GCSの用語
大切なのは、バケットとオブジェクト

詳しいことは公式の以下のリファレンスに記載されているので、簡単に説明
https://cloud.google.com/storage/docs/key-terms?hl=ja

### バケット
Google Cloud Storage 内のデータを格納するための入れ物。
ファイルサーバで言うところのディレクトリのようなもの。
この中にオブジェクトを格納する。
ただし、バケットは入れ子にできないので注意。

### オブジェクト
Google Cloud Storage 内に保存する個々のデータ。

## GCSのオブジェクトのライフサイクル管理
オブジェクトに対して様々な設定をし、管理することができる。

以下のようなことが可能になる。
> バケットにライフサイクル管理の設定を割り当てることができます。たとえば、バケット内のすべてのオブジェクトに適用されるルールを設定できます。オブジェクトがいずれかのルールの条件に一致すると、Google Cloud Storage はオブジェクトに指定された操作を自動的に実行します

引用元
https://cloud.google.com/storage/docs/lifecycle?hl=ja

例えば、「1年間経過したオブジェクトを削除する」という設定をすることができる

## 実際にやってみた
大まかな流れとしては以下になる
1. オブジェクトのライフサイクル管理の設定を記述したjsonファイルを作成する
2. gsutilツールのコマンドで、1で作成したjsonの設定をGCSに設定する

### オブジェクトのライフサイクル管理の設定を記述したjsonファイルを作成する

以下では、365日経過後に自動削除を行う設定をしている
詳しいことは以下の公式リファレンスを参照
https://cloud.google.com/storage/docs/lifecycle

``` json
{
    "lifecycle": {
        "rule": [
            {
                "action": {
                    "type": "Delete"
                },
                "condition": {
                    "age": 365,
                    "isLive": true
                }
            }
        ]
    }
}
```
`"type": "Delete" `
ライブ オブジェクトまたはアーカイブされたオブジェクトを削除する

`"age": 365 `
日数指定条件、ageで指定した日数が経過したら条件が満たされたとして、actionが発動する


### gsutilツールのコマンドで、1で作成したjsonの設定をGCSに設定する
以下のコマンドでjsonの設定をGCSに設定する

```
gsutil lifecycle set [jsonのファイル名] gs://[設定対象のGCSのバケット名]
```

詳しいことは以下の公式リファレンスを参照
https://cloud.google.com/storage/docs/managing-lifecycles


## 設定できたかどうかの確認 (2017年5月11日追記)
以下のコマンドで、GCSのバケットのライフサイクルの設定を確認することができる

```
gsutil lifecycle get gs://[設定対象のGCSのバケット名]
```
こんな感じのデータが返ってくる

```json
{"rule": [{"action": {"type": "Delete"}, "condition": {"age": 365, "isLive": true}}]}
```


## 設定後の注意
上記の手順でライフサイクル設定を行っても、更新が反映されるまでに、MAX 24 時間かかるそうなので、あれっと思うことがあるかもしれないが、そこは気長に待つしかない


## 参考にさせていただいたサイト
[gstutil](https://cloud.google.com/storage/docs/gsutil "gstutil")

[GCSの用語](https://cloud.google.com/storage/docs/key-terms?hl=ja )

[ライフサイクルの管理](https://cloud.google.com/storage/docs/managing-lifecycles)

[オブジェクトのライフサイクル管理](https://cloud.google.com/storage/docs/lifecycle#configuration)
