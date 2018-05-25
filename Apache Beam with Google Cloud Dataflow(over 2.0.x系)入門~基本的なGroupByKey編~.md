# Apache Beam with Google Cloud Dataflow(over 2.0.x系)入門~基本的なGroupByKey編~
Apache Beamの5つのCore Transformの内の1つ、GroupByKeyの基本的な使い方について記す。
CoGroupByKeyなどについては別の機会に書けたらなと思う。

Apache Beam や Cloud Dataflowの基本については[こちら](http://qiita.com/Sekky0905/items/381ed27fca9a16f8ef07)

[公式のBeam Programming Guide](https://beam.apache.org/documentation/programming-guide/#transforms-gbk)を参考に書かせていただいている。

## GroupByKeyとは
並列なreduction操作。
Map/Shuffle/Reduce-styleでいうところのShuffle。
GroupByKeyは、簡単に言うとその名の通り「KeyによってCollectionをGroup化する」Core Transform。
Keyは同じだが、valueが異なるペアが複数存在するKey-ValueなCollectionを結合して新しいCollectionを生成する。
共通なKeyを持っているデータを集約するのに役に立つ。

## multimapとuni-map

### multimap
例えば、Java, Python, GoというKeyがあったとする。
その複数の各Keyに各々Valueが数字で割り当てられている。
変換前のこのMapをmultimapという。

```
Java,1
Python,5
Go,1
Java,3
Java,2
Go,5
Python,2
Go,2
Go,9
Python,6
```

### uni-map
上記のKey-ValueなmultimapのCollectionに対してGroupByKeyを適用すると以下のような結果が得られる。

```
Java [1, 6, 8]
Python [2, 7]
Go[7, 8]
```

変換後このMapをuni-mapと呼ぶ。
一意のJava, Python, GoというKeyに対して、数字のCollectionのMapが割り当てられている。



##  Beam SDK for Java特有のKey-Valueの表し方
 Beam SDK for Javaでは、通常のJavaとは異なるKey-Valueの表し方をする。
KV<K, V>という型でkey-valueのオブジェクトを表す。

## 実際にコードを書いてみた

### 読み込むファイル

``` :sample.txt
Java,1
Python,5
Go,1
Java,3
Java,2
Go,5
Python,2
Go,2
Go,9
Python,6
```

## 実際のJavaのコード
各処理は、コードにコメントとして記載している。
理解を優先するため、メソッドチェーンを極力使用していない。
そのため、冗長なコードになっている。

```java 
import org.apache.beam.sdk.Pipeline;
import org.apache.beam.sdk.io.TextIO;
import org.apache.beam.sdk.options.PipelineOptions;
import org.apache.beam.sdk.options.PipelineOptionsFactory;
import org.apache.beam.sdk.transforms.DoFn;
import org.apache.beam.sdk.transforms.GroupByKey;
import org.apache.beam.sdk.transforms.ParDo;
import org.apache.beam.sdk.values.KV;
import org.apache.beam.sdk.values.PCollection;


/**
 * メイン
 * Created by sekiguchikai on 2017/07/12.
 */
public class Main {
    /**
     * 関数オブジェクト
     * 与えられたString str, String numを","で分割し、
     * numをInteger型に変更して、KV<String, Integer>型にする
     */
    static class SplitWordsAndMakeKVFn extends DoFn<String, KV<String, Integer>> {
        @ProcessElement
        // ProcessContextは、inputを表すobject
        // 自分で定義しなくてもBeam SDKが勝手に取ってきてくれる
        public void processElement(ProcessContext c) {
            // ","で分割
            String[] words = c.element().split(",");
            // 分割したword[0]をKに、words[1]をIntegerに変換してVにする
            c.output(KV.of(words[0], Integer.parseInt(words[1])));
        }
    }

    /**
     * 関数オブジェクト
     * KV<String, Iterable<Integer>型をString型に変更する
     */
    static class TransTypeFromKVAndMakeStringFn extends DoFn<KV<String, Iterable<Integer>>, String> {
        @ProcessElement
        public void processElement(ProcessContext c) {
            // inputをString型に変換する
            c.output(String.valueOf(c.element()));

        }

    }


    /**
     * インプットデータのパス
     */
    private static final String INPUT_FILE_PATH = "./sample.txt";
    /**
     * アウトデータのパス
     */
    private static final String OUTPUT_FILE_PATH = "./result.csv";

    /**
     * メイン
     * 理解のため、メソッドチェーンを極力使用していない
     * そのため、冗長なコードになっている
     *
     * @param args 引数
     */
    public static void main(String[] args) {
        // まずPipelineに設定するOptionを作成する
        // 今回は、ローカルで起動するため、DirectRunnerを指定する
        // ローカルモードでは、DirectRunnerがすでにデフォルトになっているため、ランナーを設定する必要はない
        PipelineOptions options = PipelineOptionsFactory.create();

        // Optionを元にPipelineを生成する
        Pipeline pipeline = Pipeline.create(options);

        // inout dataを読み込んで、そこからPCollection(パイプライン内の一連のデータ)を作成する
        PCollection<String> lines = pipeline.apply(TextIO.read().from(INPUT_FILE_PATH));

        //　与えられたString str, String numを","で分割し、numをInteger型に変更して、KV<String, Integer>型にする
        PCollection<KV<String, Integer>> kvCounter = lines.apply(ParDo.of(new SplitWordsAndMakeKVFn()));

        // GroupByKeyで、{Go, [2, 9, 1, 5]}のような形にする
　　　　　　　　　　　　　　　// GroupByKey.<K, V>create())でGroupByKey<K, V>を生成している
        PCollection<KV<String, Iterable<Integer>>> groupedWords = kvCounter.apply(
                GroupByKey.<String, Integer>create());

        // 出力のため、<KV<String, Iterable<Integer>>>型からString型に変換している
        PCollection<String> output = groupedWords.apply(ParDo.of(new TransTypeFromKVAndMakeStringFn()));

        // 書き込む
        output.apply(TextIO.write().to(OUTPUT_FILE_PATH));

        // run : PipeLine optionで指定したRunnerで実行
        // waitUntilFinish : PipeLineが終了するまで待って、最終的な状態を返す
        pipeline.run().waitUntilFinish();
    }
}

```


ちなみにメソッドチェーンを使うとこんな感じ。
だいぶすっきりした。


```java

import org.apache.beam.sdk.Pipeline;
import org.apache.beam.sdk.io.TextIO;
import org.apache.beam.sdk.options.PipelineOptionsFactory;
import org.apache.beam.sdk.transforms.DoFn;
import org.apache.beam.sdk.transforms.GroupByKey;
import org.apache.beam.sdk.transforms.ParDo;
import org.apache.beam.sdk.values.KV;


/**
 * メイン
 * Created by sekiguchikai on 2017/07/12.
 */
public class Main {
    /**
     * 関数オブジェクト
     * 与えられたString str, String numを","で分割し、
     * numをInteger型に変更して、KV<String, Integer>型にする
     */
    static class SplitWordsAndMakeKVFn extends DoFn<String, KV<String, Integer>> {
        @ProcessElement
        // ProcessContextは、inputを表すobject
        // 自分で定義しなくてもBeam SDKが勝手に取ってきてくれる
        public void processElement(ProcessContext c) {
            // ","で分割
            String[] words = c.element().split(",");
            // 分割したword[0]をKに、words[1]をIntegerに変換してVにする
            c.output(KV.of(words[0], Integer.parseInt(words[1])));
        }
    }

    /**
     * 関数オブジェクト
     * KV<String, Iterable<Integer>型をString型に変更する
     */
    static class TransTypeFromKVAndMakeStringFn extends DoFn<KV<String, Iterable<Integer>>, String> {
        @ProcessElement
        public void processElement(ProcessContext c) {
            // inputをString型に変換する
            c.output(String.valueOf(c.element()));

        }

    }


    /**
     * インプットデータのパス
     */
    private static final String INPUT_FILE_PATH = "./sample.txt";
    /**
     * アウトデータのパス
     */
    private static final String OUTPUT_FILE_PATH = "./result.csv";

    /**
     * メイン
     * 理解のため、メソッドチェーンを極力使用していない
     * そのため、冗長なコードになっている
     *
     * @param args 引数
     */
    public static void main(String[] args) {
        Pipeline pipeline = Pipeline.create(PipelineOptionsFactory.create());

        // メソッドチェーンを使った書き方
        pipeline.apply(TextIO.read().from(INPUT_FILE_PATH))
                .apply(ParDo.of(new SplitWordsAndMakeKVFn()))
                .apply(GroupByKey.<String, Integer>create())
                .apply(ParDo.of(new TransTypeFromKVAndMakeStringFn()))
                .apply(TextIO.write().to(OUTPUT_FILE_PATH));

        // run : PipeLine optionで指定したRunnerで実行
        // waitUntilFinish : PipeLineが終了するまで待って、最終的な状態を返す
        pipeline.run().waitUntilFinish();
    }
}

```

### 実行結果
以下の3つのファイルが生成される。
result.csv-00000-of-00003
result.csv-00001-of-00003
result.csv-00002-of-00003

それぞれのファイルの中身は、以下。
分散並列処理で処理が行われているので、中身が空白のファイルや、中身が1つ、2つのものがあったりとバラバラである。
また、どの内容がどのファイルに出力されるかは毎回ランダムである。

result.csv-00000-of-00003
中身なし

result.csv-00001-of-00003

```
KV{Java, [1, 3, 2]}
```

result.csv-00002-of-00003

```
KV{Go, [5, 2, 9, 1]}
KV{Python, [5, 2, 6]}
```

## 関連記事
[Apache Beam with Cloud Dataflow(over 2.0.0系)入門~基本部分~ParDoまで~ - Qiita](http://qiita.com/Sekky0905/items/381ed27fca9a16f8ef07)

[IntelliJとGradleで始めるApache Beam 2.0.x with Google Cloud Dataflow - Qiita](http://qiita.com/Sekky0905/items/40546ba36113a37a2dd3)

## 参考にさせていただいたサイト
[Beam Programming Guide](https://beam.apache.org/documentation/programming-guide/#transforms-gbk)

[GroupByKey と結合  |  Cloud Dataflow のドキュメント  |  Google Cloud Platform](https://cloud.google.com/dataflow/model/group-by-key?hl=ja)

※　[ブログ](http://www.sekky0905.com/entry/2017/10/11/Apache_Beam_with_Google_Cloud_Dataflow%28over_2_0_x%E7%B3%BB%29%E5%85%A5%E9%96%80~%E5%9F%BA%E6%9C%AC%E7%9A%84%E3%81%AAGroupByKey%E7%B7%A8~)でも同一の投稿を行っている

