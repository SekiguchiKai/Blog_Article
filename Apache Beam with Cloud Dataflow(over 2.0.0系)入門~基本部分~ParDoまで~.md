# Apache Beam with Cloud Dataflow(over 2.0.0系)入門~基本部分~ParDoまで~
何回かに分けてApache Beam with dataflowについて書こうと思う。
今回は、基本的なこと~TransformのParDoまでを書く。
最後にローカルで動作するコードも書く。

各Transformについての記事は以下。
[Apache Beam with Google Cloud Dataflow(over 2.0.x系)入門~基本的なGroupByKey編~ - Qiita](http://qiita.com/Sekky0905/items/941ad819f00390a6929e)

[Apache Beam with Google Cloud Dataflow(over 2.0.x系)入門~Combine編~ - Qiita](http://qiita.com/Sekky0905/items/4596660455a7a2af5906)

## Apache BeamとCloud Dataflow
### Apache Beamとは
バッチ処理とストリーミング処理のためのプログラミングモデル

> Implement batch and streaming data processing jobs that run on any execution engine.

(意訳) あらゆる実行エンジンで動作するバッチとストリーミングデータ処理ジョブの実装
引用元 : [Apache Beam](https://beam.apache.org/)

### Cloud Dataflowとは
GCPのApache Beamの実行環境

### Apache BeamとCloud Dataflowの関係
「Apache Beamで実装し、Cloud Dataflowで実行する」といった感じ。
ちなみに、実行環境はGoogle Cloud Dataflowのみではない。
詳しくは以下を参照
[Apache Beam Capability Matrix](https://beam.apache.org/documentation/runners/capability-matrix/)

## Batchとstreaming
Apache Beamでは、Batch処理とstreaming処理を同じような実装で扱えるようにしようとしている。ここでそもそものBatchとstreamingの定義を考えてみたい。

### Batch
> バッチ処理とは、一定期間(もしくは一定量)データを集め、まとめて一括処理を行う処理方式。または、複数の手順からなる処理において、あらかじめ一連の手順を登録しておき、自動的に連続処理を行う処理方式。

引用元 : [バッチ処理（一括処理）とは - IT用語辞典](http://e-words.jp/w/%E3%83%90%E3%83%83%E3%83%81%E5%87%A6%E7%90%86.html)

=> つまり、範囲が存在する(bounded)なもの。

### Streaming 
> 無限に発生し続けるデータを処理するよう設計されたデータ処理モデル

引用元 : [最近のストリーム処理事情振り返り](https://www.slideshare.net/SotaroKimura/ss-72769963)
詳しくは以下を参照
[最近のストリーム処理事情振り返り](https://www.slideshare.net/SotaroKimura/ss-72769963)
[【要約】The world beyond batch: Streaming 101 - Qiita](http://qiita.com/kimutansk/items/447df5795768a483caa8)

=> つまり、範囲が存在しない(unbounded)なもの。

## 重要な4つの用語
### Pipeline
データ処理タスク全体をカプセル化する。(inputデータの読み取り=>データの変換、=>outputデータの書き込み 全体)
Beam Driverのプログラムを作成するときには、このPipelineクラスの作成が必須。

### PCollection
Pipelineが動作する分散データセットを表す。このPipelineにセットするデータは、有限のデータでも無限のデータでも良い。誤解を恐れずに言えば、JavaのCollectionクラスの一種のようなものであると考えれば良いかもしれない。
基本的に、inputデータからこのPCollectionというデータセットを生成して、加工処理して、新しいPCollectionを生成して、outputに出力するというのが大まかな流れ。

### Transform
わかりやすかったので、公式からの引用
> A Transform represents a data processing operation, or a step, in your pipeline. Every Transform takes one or more PCollection objects as input, performs a processing function that you provide on the elements of that PCollection, and produces one or more output PCollection objects.

(意訳) Transformは、パイプラインのデータ処理操作またはステップを表す。全ての Transformは、1つ以上のPCollectionオブジェクトを入力として受け取り、その要素に対して提供する処理関数を実行し、PCollection1つ以上の出力PCollectionオブジェクトを生成する。
引用元 : [Beam Programming Guide](https://beam.apache.org/documentation/programming-guide/#overview)

### I/O Source and Sink
 Source :外部データからの読み込みを表す(input)
 Sink:外部データへの書き込みを表す(output)

上記の用語の説明は以下を参考にさせていただいている。
[Beam Programming Guide](https://beam.apache.org/documentation/programming-guide/#overview)

## Apache Beamの大まかな流れ
1 Pipelineを作成(この際にPipeline Runnerに依存する部分を含む実行時のオプションを指定)
2 Source APIを使用して外部データの読み込み 、それを元にPcollectionを生成
3 Transformを実施
4 Sink APIを使用して外部ソースにデータを書き込み
5 指定したPipeline Runner で実行

簡単な図にすると以下のような感じ
1  Pipelineをcreate => 2 Source APIでinput => 3 Transform => 4 Sink APIでoutput 5 Run


### 雛形を作る
### mavenの場合
以下の公式の手順に従って行う
[Java と Apache Maven を使用したクイックスタート  |  Cloud Dataflow のドキュメント  |  Google Cloud Platform](https://cloud.google.com/dataflow/docs/quickstarts/quickstart-java-maven?hl=ja)

ただし、Maven Archetype Pluginを使用して、サンプルを含んだ Maven プロジェクトを作成するときは、以下で行う(-DarchetypeVersion=2.0.0にする)

mavenのarchetype:generateついて詳しく知りたい場合はいかがわかりやすい
[Maven を使った Java project 作成方法 - Qiita](http://qiita.com/hide/items/6593f3f02c3f28e57f2d)
[Maven Archetype Plugin – archetype:generate](https://maven.apache.org/archetype/maven-archetype-plugin/generate-mojo.html)
[2. Maven 入門 (2) | TECHSCORE(テックスコア)](http://www.techscore.com/tech/Java/ApacheJakarta/Maven/2-2/)

mavenのリポジトリ
[Maven Repository: com.google.cloud.dataflow](https://mvnrepository.com/artifact/com.google.cloud.dataflow)

### gradleの場合
[IntelliJとGradleで始めるApache Beam 2.0.x with Google Cloud Dataflow - Qiita](http://qiita.com/Sekky0905/items/40546ba36113a37a2dd3)を参照

## Pipelineのoptionについて
Pipeline runner(つまり、Cloud Dataflowなどの実行環境)の設定をPipelineのoptionで設定する。これを設定する際には、コード上からプログラムによって設定するよりもコマンドラインでの引数によって設定する方が好ましい。というのも、Apache Beamの良いところの1つに、基本的にApache Beamで一回実装してしまえば、どのRunner(Apache Beamに対応する実行環境)であっても実行することができるPortabilityがあり、コード上でRunnerの設定をしてしまうと、コードによってRunnerが固定されてしまいこのPortabilityが発揮されなくなってしまうからである。(コマンドラインの引数でRunnerの設定を指定できるようにすれば、実行時に実行環境を選ぶことができ、1つのコードで様々なRunnerで実行する時に使い回すことができる)

## Transform
Transformとは、Pipelineに対する操作。
PipeLineをinputして新しいPipelineをoutputする。
Input PCollection => Transform => Output PCollection

コード的には以下のように書く

```
[Output PCollection] = [Input PCollection].apply([Transform])
```

[Beam Programming Guide](https://beam.apache.org/documentation/programming-guide/#transforms-pardo) よりコード引用

メソッドチェーンのようにもつなげることができる(厳密にはメソッドチェーンとは違うようだが…)

```
[Final Output PCollection] = [Initial Input PCollection].apply([First Transform])
.apply([Second Transform])
.apply([Third Transform])
```

[Beam Programming Guide](https://beam.apache.org/documentation/programming-guide/#transforms-pardo) よりコード引用

Apache Beamでは、汎用的な処理フレームワークを提供していて、開発者は処理ロジックを関数オブジェクトで定義する。
この関数オブジェクトはuser codeと呼ばれる。

Apache Beamでは以下の5つのCore TransformというTransformを提供している。
今回は、ParDo のみ説明を行う
* ParDo 
* Using GroupByKey
* Using Combine
* Using Flatten and Partition

## ParDo
一般的な並列処理をする時に使用する。
 Map/Shuffle/Reduce で言うところの Mapper のようなもの。
inputとしてPCollectionが入力され、処理を行いinputされたCollectionを加工してoutputとして0~複数の新しいPCollectionを出力する。

ParDoで使用する処理を自分で実装する場合、DoFnのサブクラスを作成する。

こんな感じです。

```java
// まずは外部ソースから読み込んで、PCollectionを生成する
PCollection<String> inputData = // ここで外部ソースから読み込みを行う;

// static classとして関数オブジェクトを定義 
// DoFnの左側のinputの型を、右側にoutputの型を型パラメータとして定義する
// 必ず、DoFnをextendsする 
static class FilterEvenFn extends DoFn<String, Integer> {
  // 実際の処理ロジックをアノテーションで宣言する
  @ProcessElement
  // 実際の処理ロジックは、processElementメソッドに記述する
  // 引数のProcessContextを利用してinputやoutputを行う 
  public void processElement(ProcessContext c) {
    // ProcessContextからinput elementを取得
    int num = Integer.parseInt(c.element());
    // input elementを使用した処理
    if (num % 2 == 0) {
        // ProcessContextを使用して出力
        c.output(String.valueOf(num));
    }
  }
}
// 
PCollection<Integer> evenData = inputData.apply(
    ParDo
    .of(new FilterEvenFn()));  
```

## データの読み込み
INPUT_FILE_PATHの場所で読み込むリソースを指定する
Runnerの環境に依存している場合は、それに従う(例えば、Google Cloud PlatformのGoogle Cloud Storageなど )

```java
PCollection<String> newPCollection = pipeline.apply(TextIO.read().from(INPUT_FILE_PATH));
```

## データの書き込み
Runnerの環境に依存している場合は、それに従う(例えば、Google Cloud PlatformのBigQueryなど )

```java
pCollection.apply(TextIO.write().to(OUTPUT_FILE_PATH));
```

## Pipelineの実行
PipeLine optionで指定したRunnerでPipelineを実行する

```java
pipeline.run();
```

## 実際に簡単なコードを書いてみた
今回はGoogle Cloud Platformのようなクラウド上の実行環境ではなく、自分のローカル環境で動作するような簡単なコードを書いてみた。

以下のようなファイルを偶数のみ抽出する。

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
```

個々の処理はコードにコメントとして記述している。
理解を優先するため、メソッドチェーンを極力使用していない。
そのため、冗長なコードになっている。


```java

package com.company;

import org.apache.beam.sdk.Pipeline;
import org.apache.beam.sdk.io.TextIO;
import org.apache.beam.sdk.options.PipelineOptions;
import org.apache.beam.sdk.options.PipelineOptionsFactory;
import org.apache.beam.sdk.transforms.DoFn;
import org.apache.beam.sdk.transforms.ParDo;
import org.apache.beam.sdk.values.PCollection;


/**
 * メイン
 */
public class Main {
    // DoFnを実装したクラス
    // DoFnの横の<T,T>でinputとoutputの方の定義を行う
    static class FilterEvenFn extends DoFn<String, String> {
        // 実際の処理ロジックにはこのアノテーションをつける
        @ProcessElement
        // 実際の処理ロジックは、processElementメソッドに記述する
        // 引数のProcessContextを利用してinputやoutputを行う
        public void processElement(ProcessContext c) {
            // ProcessContextからinput elementを取得
            int num = Integer.parseInt(c.element());
            // input elementを使用した処理
            if (num % 2 == 0) {
                // ProcessContextを使用して出力
                c.output(String.valueOf(num));
            }
        }
    }

    // インプットデータのパス
    private static final String INPUT_FILE_PATH = "./dataflow_number_test.csv";
    // アウトデータのパス
    private static final String OUTPUT_FILE_PATH = "./sample.csv";

    public static void main(String[] args) {
        // まずPipelineに設定するOptionを作成する
        // 今回は、ローカルで起動するため、DirectRunnerを指定する
        // ローカルモードでは、DirectRunnerがすでにデフォルトになっているため、ランナーを設定する必要はない
        PipelineOptions options = PipelineOptionsFactory.create();

        // Optionを元にPipelineを生成する
        Pipeline pipeline = Pipeline.create(options);

        // inout dataを読み込んで、そこからPCollection(パイプライン内の一連のデータ)を作成する
        PCollection<String> inputData = pipeline.apply(TextIO.read().from(INPUT_FILE_PATH));

        // 処理
        PCollection<String> evenData = inputData.apply(ParDo.of(new FilterEvenFn()));
        // 書き込む
        evenData.apply(TextIO.write().to(OUTPUT_FILE_PATH));

        // run : PipeLine optionで指定したRunnerで実行
        // waitUntilFinish : PipeLineが終了するまで待って、最終的な状態を返す
        pipeline.run().waitUntilFinish();
    }
}
```

メソッドチェーンを使用した場合

```java

import org.apache.beam.sdk.Pipeline;
import org.apache.beam.sdk.io.TextIO;
import org.apache.beam.sdk.options.PipelineOptionsFactory;
import org.apache.beam.sdk.transforms.DoFn;
import org.apache.beam.sdk.transforms.ParDo;


/**
 * メイン
 * Created by sekiguchikai on 2017/07/05.
 */
public class Main {
    // DoFnを実装したクラス
    // DoFnの横の<T,T>でinputとoutputの方の定義を行う
    static class FilterEvenFn extends DoFn<String, String> {
        // 実際の処理ロジックにはこのアノテーションをつける
        @ProcessElement
        // 実際の処理ロジックは、processElementメソッドに記述する
        // 引数のProcessContextを利用してinputやoutputを行う
        public void processElement(ProcessContext c) {
            System.out.print(c.element());
            // ProcessContextからinput elementを取得
            int num = Integer.parseInt(c.element());
            // input elementを使用した処理
            if (num % 2 == 0) {
                System.out.println("ifの結果" + num);
                // ProcessContextを使用して出力
                c.output(String.valueOf(num));
            }
        }
    }

    // インプットデータのパス
    private static final String INPUT_FILE_PATH = "./dataflow_number_test.csv";
    // アウトデータのパス
    private static final String OUTPUT_FILE_PATH = "./sample.csv";

    public static void main(String[] args) {
        // Optionを元にPipelineを生成する
        Pipeline pipeline = Pipeline.create(PipelineOptionsFactory.create());

        // メソッドチェーンを使用
        pipeline.apply(TextIO.read().from(INPUT_FILE_PATH))
                .apply(ParDo.of(new FilterEvenFn()))
                // 書き込む
                .apply(TextIO.write().to(OUTPUT_FILE_PATH));

        // run : PipeLine optionで指定したRunnerで実行
        // waitUntilFinish : PipeLineが終了するまで待って、最終的な状態を返す
        pipeline.run().waitUntilFinish();
    }
}
```


## 実行結果
以下のファイルが生成される
sample.csv-00000-of-00004
sample.csv-00001-of-00004
sample.csv-00002-of-00004
sample.csv-00003-of-00004

それぞれのファイルの中身は以下。(分散されて並列で処理されているので、バラバラ)

sample.csv-00000-of-00004

```
4
12
```


sample.csv-00001-of-00004

```
8
```


sample.csv-00002-of-00004

```
2
10
14
```


sample.csv-00003-of-00004

```
6
```


このファイルをBigQueryに突っ込むなりして分析したりするといいかもしれない。

## 関連記事
[Apache Beam with Google Cloud Dataflow(over 2.0.x系)入門~基本的なGroupByKey編~ - Qiita](http://qiita.com/Sekky0905/items/941ad819f00390a6929e)

[IntelliJとGradleで始めるApache Beam 2.0.x with Google Cloud Dataflow - Qiita](http://qiita.com/Sekky0905/items/40546ba36113a37a2dd3)

[Apache Beam with Google Cloud Dataflow(over 2.0.x系)入門~Combine編~ - Qiita](http://qiita.com/Sekky0905/items/4596660455a7a2af5906)


## 参考にさせていただいたサイト

[バッチ処理（一括処理）とは - IT用語辞典](http://e-words.jp/w/%E3%83%90%E3%83%83%E3%83%81%E5%87%A6%E7%90%86.html)

 [Apache Beam](https://beam.apache.org/)

[最近のストリーム処理事情振り返り](https://www.slideshare.net/SotaroKimura/ss-72769963)

[【要約】The world beyond batch: Streaming 101 - Qiita](http://qiita.com/kimutansk/items/447df5795768a483caa8)

[【要約】The world beyond batch: Streaming 102 - Qiita](http://qiita.com/kimutansk/items/c6b7cccd976ff00cb1fd)


[Beam Programming Guide](https://beam.apache.org/documentation/programming-guide/#overview)

[Java と Apache Maven を使用したクイックスタート  |  Cloud Dataflow のドキュメント  |  Google Cloud Platform](https://cloud.google.com/dataflow/docs/quickstarts/quickstart-java-maven?hl=ja)

[Maven を使った Java project 作成方法 - Qiita](http://qiita.com/hide/items/6593f3f02c3f28e57f2d)
[Maven Archetype Plugin – archetype:generate](https://maven.apache.org/archetype/maven-archetype-plugin/generate-mojo.html)
[2. Maven 入門 (2) | TECHSCORE(テックスコア)](http://www.techscore.com/tech/Java/ApacheJakarta/Maven/2-2/)

[Maven Repository: com.google.cloud.dataflow](https://mvnrepository.com/artifact/com.google.cloud.dataflow)

[パイプラインの実行パラメータを指定する  |  Cloud Dataflow のドキュメント  |  Google Cloud Platform](https://cloud.google.com/dataflow/pipelines/specifying-exec-params?hl=ja)

※ ブログでも同一の投稿を行っている
[Apache Beam with dataflow入門~基本部分~ParDoまで~ ](http://www.sekky0905.com/entry/2017/07/12/Apache_Beam_with_dataflow%E5%85%A5%E9%96%80~%E5%9F%BA%E6%9C%AC%E9%83%A8%E5%88%86~ParDo%E3%81%BE%E3%81%A7~)
