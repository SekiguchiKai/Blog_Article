# Apache Beam with Google Cloud Dataflow(over 2.0.x系)入門~Combine~

Apache Beamの5つのCore Transformの内の1つ、Combineの基本的な使い方について記す。
他のCore TransformやそもそものApache Beam 2.0.xの基本的な話は以下に記述している。

[IntelliJとGradleで始めるApache Beam 2.0.x with Google Cloud Dataflow - Qiita](http://qiita.com/Sekky0905/items/40546ba36113a37a2dd3)

[Apache Beam with Cloud Dataflow(over 2.0.0系)入門~基本部分~ParDoまで~ - Qiita](http://qiita.com/Sekky0905/items/381ed27fca9a16f8ef07)

[Apache Beam with Google Cloud Dataflow(over 2.0.x系)入門~基本的なGroupByKey編~ - Qiita](http://qiita.com/Sekky0905/items/941ad819f00390a6929e)

なお、本記事は以下の2つの公式ドキュメントを参考に記述している。

[Beam Programming Guide](https://beam.apache.org/documentation/programming-guide/#transforms-combine)

[コレクションと値の結合  |  Cloud Dataflow のドキュメント  |  Google Cloud Platform](https://cloud.google.com/dataflow/model/combine?hl=ja)


## Combineの2つの役割
Combineは、PCollection内に存在する各要素(各データ)を結合したり、マージする。
Map/Shuffle/ReduceでいうところのReduceのようなものだと認識している。

Combineの仕方は大きく分けて2つ存在する。
「1つのPCollection内に存在する要素を結合して、1つの値を生成する方法」と「KeyによってGroup化されたPCollectionのValue部分の各要素を結合して、1つの値を生成する方法」である。
以下、各々の方法を記したい。

# 1つのPCollection内に存在する要素を結合して、1つの値を生成する方法

## 1つのPCollection内に存在する要素を結合して、1つの値を生成する方法とは
PCollection内の各要素を結合する。
=>これはParDoとの違いに注意する必要がある。
ParDoは、PCollection内の各々の要素に対して何らかの処理を行う。
Combineは、PCollection内の各要素を結合する。

例えば、PCollection内に存在する要素を結合して、1つの値を生成する場合がこれ。

```java

PCollection<Integer> sum = pCollection.apply(Sum.integersGlobally());
```

一見、Combineが存在しないように見えるが、Sum.integersGlobally()が、 Combine.globallyをwrapしている。実際のSum.integersGlobally()は以下。

```java

public static Combine.Globally<Integer, Integer> integersGlobally() {
  return Combine.globally(Sum.ofIntegers());}
```
参考
[API リファレンス](https://beam.apache.org/documentation/sdks/javadoc/2.0.0/index.html?org/apache/beam/sdk/transforms/Combine.html)

## withoutDefaults()
空のPCollectionがinoutとして与えられた場合に、emptyを返したいなら withoutDefaults()をつける。

```java

PCollection<Integer> sum = integerPCollection.apply(Sum.integersGlobally().withoutDefaults());
```


## Global Windowの場合と非Global Windowの場合の動作の違い
 Global Windowの場合には、1 つの項目を含んだ PCollection を返すことがデフォルトの動作になっている。

一方、非Global Windowの場合、上記のようなデフォルトの動作はしない。
Combineを使用する際に、Optionを指定する。
公式がわかりやすかったので、以下引用。(本投稿執筆時には、[Apache Beam 2.0.x](https://beam.apache.org/documentation/programming-guide/#transforms-combine)の方のDocumentにはまだこの記載が存在しなかったため、[Google Cloud Dataflow1.9](https://cloud.google.com/dataflow/model/combine?hl=ja)の方の公式ドキュメントから引用させていただいている)

>.withoutDefaults を指定する。この場合、入力 PCollection 内の空のウィンドウは、出力>コレクションでも空になります。

>.asSingletonView を指定する。この場合、出力は直ちに PCollectionView へと変換されます。これは、それぞれの空ウィンドウが副入力として使用される場合のデフォルト値になります。通常、このオプションは、パイプラインの Combine の結果が後にパイプライン内で副入力として使用される場合にのみ、使用する必要があります。

引用元 : [コレクションと値の結合  |  Cloud Dataflow のドキュメント  |  Google Cloud Platform](https://cloud.google.com/dataflow/model/combine?hl=ja)

## 実際にコードを書いてみた
各処理は、コードにコメントとして記載している。
理解を優先するため、メソッドチェーンを極力使用していない。
そのため、冗長なコードになっている。

```java
package com.company;

import org.apache.beam.sdk.Pipeline;
import org.apache.beam.sdk.io.TextIO;
import org.apache.beam.sdk.options.PipelineOptionsFactory;
import org.apache.beam.sdk.transforms.DoFn;
import org.apache.beam.sdk.transforms.ParDo;
import org.apache.beam.sdk.transforms.Sum;
import org.apache.beam.sdk.values.PCollection;

/**
 * メインクラス
 */
public class Main {
    /**
     * 関数型オブジェクト
     * String => Integerの型変換を行う
     */
    static class TransformTypeFromStringToInteger extends DoFn<String, Integer> {
        @ProcessElement
        public void processElement(ProcessContext c) {
            // 要素をString=>Integerに変換して、output
            c.output(Integer.parseInt(c.element()));
        }
    }

    /**
     * 関数型オブジェクト
     * Integer =>Stringの型変換を行う
     */
    static class TransformTypeFromIntegerToString extends DoFn<Integer, String> {
        @ProcessElement
        public void processElement(ProcessContext c) {
            // 要素をString=>Integerに変換して、output
            System.out.println(c.element());
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
    private static final String OUTPUT_FILE_PATH = "./result.txt";

    /**
     * 理解のためにメソッドチェーンは極力使用しない
     * そのため冗長な箇所がある
     * メインメソッド
     *
     * @param args
     */
    public static void main(String[] args) {
        // optionを指定して、Pipelineを生成する
        Pipeline pipeline = Pipeline.create(PipelineOptionsFactory.create());

        System.out.println("a");
        // ファイルから読み込み
        PCollection<String> lines = pipeline.apply(TextIO.read().from(INPUT_FILE_PATH));
        // 読み込んだ各データをString => Integerに変換
        PCollection<Integer> integerPCollection = lines.apply(ParDo.of(new TransformTypeFromStringToInteger()));
        // Combine.GloballyでPCollectionの各要素を合計
        // 空のPCollectionの場合、emptyを返したいなら => PCollection<Integer> sum = integerPCollection.apply(Sum.integersGlobally().withoutDefaults());
        PCollection<Integer> sum = integerPCollection.apply(Sum.integersGlobally().withoutDefaults());
        // PCollection<Integer> sumをInteger => Stringに変換
        PCollection<String> sumString = sum.apply(ParDo.of(new TransformTypeFromIntegerToString()));
        // ファイルに書き込み
        sumString.apply(TextIO.write().to(OUTPUT_FILE_PATH));

        // 実行
        pipeline.run().waitUntilFinish();
    }
}

```

### 実施にコードを書いてみた（メソッドチェーンを使ったver）
だいぶすっきりした

```java
package com.company;

import org.apache.beam.sdk.Pipeline;
import org.apache.beam.sdk.io.TextIO;
import org.apache.beam.sdk.options.PipelineOptionsFactory;
import org.apache.beam.sdk.transforms.DoFn;
import org.apache.beam.sdk.transforms.ParDo;
import org.apache.beam.sdk.transforms.Sum;


/**
 * メインクラス
 */
public class Main {
    /**
     * 関数型オブジェクト
     * String => Integerの型変換を行う
     */
    static class TransformTypeFromStringToInteger extends DoFn<String, Integer> {
        @ProcessElement
        public void processElement(ProcessContext c) {
            // 要素をString=>Integerに変換して、output
            c.output(Integer.parseInt(c.element()));
        }
    }

    /**
     * 関数型オブジェクト
     * Integer =>Stringの型変換を行う
     */
    static class TransformTypeFromIntegerToString extends DoFn<Integer, String> {
        @ProcessElement
        public void processElement(ProcessContext c) {
            // 要素をString=>Integerに変換して、output
            System.out.println(c.element());
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
    private static final String OUTPUT_FILE_PATH = "./result.txt";

    /**
     * メインメソッド
     *
     * @param args
     */
    public static void main(String[] args) {
        // Pipeline生成
        Pipeline pipeline = Pipeline.create(PipelineOptionsFactory.create());

        // 処理部分
        pipeline.apply(TextIO.read().from(INPUT_FILE_PATH))
                .apply(ParDo.of(new TransformTypeFromStringToInteger()))
                .apply(Sum.integersGlobally().withoutDefaults())
                .apply(ParDo.of(new TransformTypeFromIntegerToString()))
                .apply(TextIO.write().to(OUTPUT_FILE_PATH));

        // 実行
        pipeline.run().waitUntilFinish();
    }
}
```

### 読み込んだファイル

```:samole.txt
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
```

### 実行結果
result.txt-00000-of-00001　が出力される
result.txt-00000-of-00001の中身は

```
55
```

やっていることは、

```
10
Σk
k = 1
```

みたいなもん。


## PerKey
GroupByKeyを行うと K,V(IterableなCollection)になる。
例えば、以下のようになる。

```
Java [1, 2, 3]
```

CombineのPerKeyは、このK,V[IterableなCollection]のV[IterableなCollection]部分をKey毎に結合する。なので、例えば上記のGroupByKey後のK,V(IterableなCollection)をCombine PerKeyを行うと以下のようになる。

```
Java [6]
```

K,V(IterableなCollection)の,V(IterableなCollection)の要素がすべて結合された。


## 実際にコードを書いてみた
各処理は、コードにコメントとして記載している。
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
import org.apache.beam.sdk.transforms.Sum;
import org.apache.beam.sdk.values.KV;
import org.apache.beam.sdk.values.PCollection;

/**
 * メイン
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
    static class TransTypeFromKVAndMakeStringFn extends DoFn<KV<String, Integer>, String> {
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
    private static final String COMBINE_OUTPUT_FILE_PATH = "./src/main/resources/combine_result/result.csv";

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

        // Combine PerKey は、オペレーションの一部として GroupByKey 変換を実行する
        PCollection<KV<String, Integer>> sumPerKey = kvCounter
                .apply(Sum.integersPerKey());
        
        // PCollectionをファイル出力可能な形に変換する
        PCollection<String> output = sumPerKey.apply(ParDo.of(new TransTypeFromKVAndMakeStringFn()));

        // 書き込む
        output.apply(TextIO.write().to(COMBINE_OUTPUT_FILE_PATH));

        // run : PipeLine optionで指定したRunnerで実行
        // waitUntilFinish : PipeLineが終了するまで待って、最終的な状態を返す
        pipeline.run().waitUntilFinish();
    }


}
```

### 実施にコードを書いてみた（メソッドチェーンを使ったver）
だいぶすっきりした

```java
package com.company;

import org.apache.beam.sdk.Pipeline;
import org.apache.beam.sdk.io.TextIO;
import org.apache.beam.sdk.options.PipelineOptionsFactory;
import org.apache.beam.sdk.transforms.DoFn;
import org.apache.beam.sdk.transforms.ParDo;
import org.apache.beam.sdk.transforms.Sum;
import org.apache.beam.sdk.values.KV;

/**
 * メイン
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
    static class TransTypeFromKVAndMakeStringFn extends DoFn<KV<String, Integer>, String> {
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
    private static final String COMBINE_OUTPUT_FILE_PATH = "./src/main/resources/combine_result/result.csv";

    /**
     * メイン
     * @param args 引数
     */
    public static void main(String[] args) {
        Pipeline pipeline = Pipeline.create(PipelineOptionsFactory.create());
        pipeline
                .apply(TextIO.read().from(INPUT_FILE_PATH))
                .apply(ParDo.of(new SplitWordsAndMakeKVFn()))
                .apply(Sum.integersPerKey())
                .apply(ParDo.of(new TransTypeFromKVAndMakeStringFn()))
                .apply(TextIO.write().to(COMBINE_OUTPUT_FILE_PATH));
        pipeline.run().waitUntilFinish();
    }


}
```


### 読み込んだファイル

```:samole.txt
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

### 実行結果
以下の3つのファイルが生成される。
result.csv-00000-of-00003
result.csv-00001-of-00003
result.csv-00002-of-00003

それぞれのファイルの中身は、以下。
分散並列処理で処理が行われているので、どの内容がどのファイルに出力されるかは毎回ランダムである。

result.csv-00000-of-00003

```
KV{Python, 13}
```

result.csv-00001-of-00003

```
KV{Java, 6}
```
result.csv-00002-of-00003

```
KV{Go, 17}
```

## 関連記事
[IntelliJとGradleで始めるApache Beam 2.0.x with Google Cloud Dataflow - Qiita](http://qiita.com/Sekky0905/items/40546ba36113a37a2dd3)

[Apache Beam with Cloud Dataflow(over 2.0.0系)入門~基本部分~ParDoまで~ - Qiita](http://qiita.com/Sekky0905/items/381ed27fca9a16f8ef07)

[Apache Beam with Google Cloud Dataflow(over 2.0.x系)入門~基本的なGroupByKey編~ - Qiita](http://qiita.com/Sekky0905/items/941ad819f00390a6929e)


## 参考にさせていただいたサイト
[Beam Programming Guide](https://beam.apache.org/documentation/programming-guide/#transforms-combine)
[コレクションと値の結合  |  Cloud Dataflow のドキュメント  |  Google Cloud Platform](https://cloud.google.com/dataflow/model/combine?hl=ja)
[API リファレンス](https://beam.apache.org/documentation/sdks/javadoc/2.0.0/index.html?org/apache/beam/sdk/transforms/Combine.html)

※　[ブログ](http://www.sekky0905.com/entry/2017/10/11/Apache_Beam_with_Google_Cloud_Dataflow%28over_2_0_x%E7%B3%BB%29%E5%85%A5%E9%96%80~Combine~)でも同一の投稿を行っている。
