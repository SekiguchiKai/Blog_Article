# Javaのシリアライズ(直列化)メモ
今更だが、Javaのインスタンスをファイルなどに保存するのに便利なシリアライズ(直列化)についてのメモ。

## シリアライズとデシリアライズ
### シリアライズ（直列化）
Javaのインスタンスをバイト列として出力すること。

### デシリアライズ
バイト列をJavaのインスタンスに復元すること。

## 何が嬉しいのか
この機能を使うと、簡単にインスタンスを外部記憶装置などに保存し、インスタンスの情報を永続化することができる。
何か特別な処理をしたり、ある形式に沿ってインスタンスを保存したりするような面倒なことを省くことができる。

## java.io.Serializableインターフェース
シリアライズを行いたいインスタンスの元となるクラスでは、java.io.Serializableインタフェースを実装する必要がある。
この、Serializableインタフェースには、フィールド及びメソッドが存在せず、ただただ、シリアライズ（直列化）可能ですよということを示すことのみを提供する。

参考 : [Serializable (Java Platform SE 8 )](https://docs.oracle.com/javase/jp/8/docs/api/java/io/Serializable.html)

## ObjectOutputStreamクラスとObjectInputStreamクラス

###  ObjectOutputStreamクラス
> ObjectOutputStream は、基本データ型と Java オブジェクトのグラフを OutputStream に書き込みます。

引用元 : [ObjectOutputStream (Java Platform SE 6)](https://docs.oracle.com/javase/jp/6/api/java/io/ObjectOutputStream.html)

=> 出力用のStreamに基本データ型とオブジェクトのグラフを書き込む


###  ObjectIntputStreamクラス
> ObjectOutputStreamを使って作成されたプリミティブ・データとプリミティブ・オブジェクトを直列化復元します

引用元 : [ObjectInputStream (Java Platform SE 8 )](https://docs.oracle.com/javase/jp/8/docs/api/java/io/ObjectInputStream.html)

=> ObjectOutputStreamを使って書き込んだものを復元する

## serialVersionUID
あるクラスをシリアライズ化して、ファイルとして永続をした後、そのクラスのフィルールドなど変更したりした時に、変更後のJavaのクラスとシリアライズされたバイト列から再度でシリアライズされた際に復元されるインスタンスは、矛盾が生じる可能性がある。
そうならないために、このserialVersionUIDにバージョンとして値を与えて、コードに変更があった時には、この値を毎回変えるというふうにする。
そうすることで、復元元と復元先のこのserialVersionUIDの値が一緒だったら、同じ世代のインスタンスなので、矛盾が生じることなく、復元を行うことができると保証されるわけである。

よく、何かのシステムを作ったりする時や、プログラミング言語でもバージョニングということを行うことがあると思うが、それに近い。
例えば、Java1.8ならStream APIは存在するけど、それ未満だと存在しないよみたいな感じで。

参考 : [難解なSerializableという仕様について俺が知っていること、というか俺の理解 - 都元ダイスケ IT-PRESS](http://d.hatena.ne.jp/daisuke-m/20100414/1271228333)

参考 : 中山 清喬(2014/9/22)『スッキリわかるJava入門 実践編 第2版 スッキリわかるシリーズ』 インプレスジャパン

## 実装してみた
### 全体の流れ
1. Languageクラスをインスタンス化する
2. ファイル名を元にFileOutputStreamクラスのインスタンスを生成する
3. FileOutputStreamクラスを元に、ObjectOutputStreamクラスを生成する
4. ObjectOutputStream.writeObjectでファイルに書き込みを行う

1. ファイル名を元にFileInputStreamクラスのインスタンスを生成する
2. FileInputStreamクラスを元に、ObjectInputStreamクラスを生成する
3. ObjectInputStream.readObjectでインスタンスを復元する

図にするとこんな感じ

![serialize.png](https://qiita-image-store.s3.amazonaws.com/0/145611/cc340522-0562-424a-2be9-ca2e3e367ae5.png)






### コード

```java:Main.java
package com.company;

import java.io.*;

/**
 * メインクラス
 */
public class Main {
    /**
     * メインメソッド
     *
     * @param args コマンドライン引数
     */
    public static void main(String args[]) {
        String fileName = "lang.txt";

        // Languageクラスをインスタンス化
        Language language = new Language("Java", "Static");
        try {
            Main.Serialize(language, fileName);
        } catch (IOException e) {
            e.printStackTrace();
        }

        try {
            Language deSirializedLanguage = Main.DeSerialize(fileName);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }

        System.out.println("デシリアライズされた、Languageの名前は" + language.getName() + "タイプは" + language.getType());
    }

    /**
     * オブジェクトのシリアライズを行う
     *
     * @param language Languageクラスのインスタンス
     * @param fileName ファイル名
     * @throws IOException IO例外
     */
    private static void Serialize(Language language, String fileName) throws IOException {
        // わかりやすくするために冗長に書いている
        // ファイルを指定して、FileOutputStreamを生成
        FileOutputStream fileOutputStream = new FileOutputStream(fileName);
        // OutputStreamに基本データ型とオブジェクトのグラフをOutputStreamに書き込む
        // 今回は、FileOutputStreamに書き込む
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(fileOutputStream);

        // 特定のクラスのオブジェクトの状態をストリームに書き込む
        objectOutputStream.writeObject(language);
        objectOutputStream.flush();

        objectOutputStream.close();
    }

    /**
     * オブジェクトのデシリアライズを行う
     *
     * @return Languageクラスのインスタンス
     */
    private static Language DeSerialize(String fileName) throws IOException, ClassNotFoundException {
        // わかりやすくするために冗長に書いている
        // ファイルを指定して、FileInputStreamを生成
        FileInputStream fileInputStream = new FileInputStream(fileName);
        // 事前にObjectOutputStreamを使って作成されたプリミティブ・データとプリミティブ・オブジェクトを直列化復元する
        ObjectInputStream objectInputStream = new ObjectInputStream(fileInputStream);

        // 対応するwriteObjectメソッドによってストリームに書き込まれたデータを使用する特定のクラスのオブジェクトの状態を読み込み、復元する
        Language language = (Language) objectInputStream.readObject();

        objectInputStream.close();

        return language;
    }
}
```

```java:Language.java
package com.company;

import java.io.Serializable;

/**
 * プログラミング言語を表す
 * Serializableインターフェースを実装する
 */
public class Language implements Serializable {

    /**
     * シリアルバージョンUID
     */
    private static final long serialVersionUID = 1L;

    /**
     * 言語の名前
     */
    private String name;
    /**
     * 言語のタイプ
     */
    private String type;

    Language(String name, String type) {
        this.name = name;
        this.type = type;
    }

    /**
     * 名前
     *
     * @return nam　言語の名前
     */
    public String getName() {
        return name;
    }

    /**
     * typeのゲッター
     *
     * @return type 言語のタイプ
     */
    public String getType() {
        return type;
    }
}
```

### 実行結果
```
デシリアライズされた、Languageの名前はJavaタイプはStatic
```

## 参考文献
中山 清喬(2014/9/22)『スッキリわかるJava入門 実践編 第2版 スッキリわかるシリーズ』 インプレスジャパン

[Serializable (Java Platform SE 8 )](https://docs.oracle.com/javase/jp/8/docs/api/java/io/Serializable.html)

[ObjectOutputStream (Java Platform SE 6)](https://docs.oracle.com/javase/jp/6/api/java/io/ObjectOutputStream.html)

[ObjectInputStream (Java Platform SE 8 )](https://docs.oracle.com/javase/jp/8/docs/api/java/io/ObjectInputStream.html)

[難解なSerializableという仕様について俺が知っていること、というか俺の理解 - 都元ダイスケ IT-PRESS](http://d.hatena.ne.jp/daisuke-m/20100414/1271228333)

[【Java】Serializableの基本（シリアライズ・直列化） - TASK NOTES](http://www.task-notes.com/entry/20150925/1443150000)

[オブジェクトのシリアライズとデシリアライズ - Java 入門](http://java.keicode.com/lang/io-object-serialize.php)

[【Java】Serializableの基本（シリアライズ・直列化） - TASK NOTES](http://www.task-notes.com/entry/20150925/1443150000)

※ブログでも同一の投稿を行っている
[Javaのシリアライズ(直列化)メモ](http://www.sekky0905.com/entry/2017/08/15/Java%E3%81%AE%E3%82%B7%E3%83%AA%E3%82%A2%E3%83%A9%E3%82%A4%E3%82%BA%28%E7%9B%B4%E5%88%97%E5%8C%96%29%E3%83%A1%E3%83%A2)
