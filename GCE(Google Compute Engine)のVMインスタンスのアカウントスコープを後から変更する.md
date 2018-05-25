# GCE(Google Compute Engine)のVMインスタンスのアカウントスコープを後から変更する
GCEのVMインスタンスでアカウントスコープを途中で変更したい場合、少し工夫する必要があったのでメモ。

## アクセス スコープとは
それぞれのGoogle Cloud Platform上のサービスのAPIに対して、インスタンスがアクセスを許可されている範囲のこと。
Google Cloud Platform上の各サービスのAPIに対して、インスタンスがアクセス可能かどうかを「有効」/「無効」で表している。
これが有効のサービスのみ、インスタンスはそのサービスのAPIにアクセスすることができる。
https://cloud.google.com/compute/docs/access/service-accounts?hl=ja

## アクセススコープを変更する

### アクセススコープを変更するには
ここからが本記事の本題だが、アクセススコープは、インスタンスの作成時か、インスタンスをシャットダウンしている状態でしか編集できないようだ。

そのため、「そのインスタンスで当初BigQueryのAPIを使う予定がなかったので、アクセススコープは、『無効』にしたが、後にやっぱりBigQueryのAPIを使いたくなったので、アクセススコープを『有効』にしたい」
という場合は、前述したが以下の2つの方法しかない。(他にあったら教えてください…)
1.  インスタンスを作成し直す
2. インスタンスをシャットダウンして変更する                                                                                   

自分は当初それを知らなかったため、VMインスタンスのコンソール画面で何回編集を押してもアクセススコープが変更できなくて、泣きそうになりました。

今回は、「2.インスタンスをシャットダウンして変更する」を行い、アクセススコープを変更する。

### アクセススコープを変更するための手順

アクセススコープを変更するためには以下の手順が必要
1. アクセススコープを変更したいVMインスタンスをシャットダウンする
2. gcloudコマンドで、アクセス権を変更する
3. インスタンスを再起動する

### 実際にやってみる
1.アクセススコープを変更したいVMインスタンスをシャットダウンする
以下の公式ドキュメントを参考にVMインスタンスをシャットダウンする
https://cloud.google.com/compute/docs/instances/stopping-or-deleting-an-instance

2.以下のcloudコマンドで、アクセス権を変更する

```
gcloud beta compute instances set-scopes projects/プロジェクト名/zones/ゾーン/instances/インスタンス名  --service-account サービスアカウントのメール <—no-scopes(無効にしたい場合) か --scopes(有効にしたい場合)>  変更するスコープ
```

「変更するスコープ」は、以下の一覧にあるurlかエイリアスを使用する
https://cloud.google.com/sdk/gcloud/reference/beta/compute/instances/set-scopes

3インスタンスを再起動する
https://cloud.google.com/compute/docs/instances/restarting-an-instance#restarting_a_stopped_instance


### 実際にやってみた
実際にインスタンス作成時には「無効」であったBigqueryのアクセススコープを「有効」に変更する。

「1. アクセススコープを変更したいVMインスタンスをシャットダウンする」と「3. インスタンスを再起動する」は、省略

gcloudを実行
```
gcloud beta compute instances set-scopes projects/プロジェクト名/zones/ゾーン/instances/インスタンス名  --service-account サービスアカウントのメール --scopes https://www.googleapis.com/auth/bigquery
```

結果
VMインスタンスのコンソールの「Cloud APIアクセス範囲」のBigQueryが「無効」から「有効」になった。

## 参考にさせていただいたサイト


https://cloud.google.com/compute/docs/access/service-accounts?hl=ja

https://cloud.google.com/compute/docs/instances/stopping-or-deleting-an-instance

https://cloud.google.com/sdk/gcloud/reference/beta/compute/instances/set-scopes

https://cloud.google.com/compute/docs/instances/restarting-an-instance#restarting_a_stopped_instance

http://stackoverflow.com/questions/31855049/how-to-change-the-scope-of-compute-engine-service-account-to-write-data-on-googl
