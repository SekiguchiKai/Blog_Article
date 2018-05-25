# Python3とJavaでクラスを書いてみた

静的言語ばかりやってきたが、機械学習とかやりたいので、Pythonの勉強を始めることにした。
これまでやってきたGo、Java、TypeScriptとはなかなか違うところが多くてカルチャーショックを受けている...
とは言え、新しい言語を学ぶのは心が躍るものだ。

とりあえず、Pythonのclass構文を忘れないようにJavaとPythonのclassを比較できるように両方の言語で同じクラスとその子クラスを書いてみた。


## やっていることの説明
どちらもPhoneクラスを作成する。
次にその子クラスであるSmartPhoneクラスを作成する。
両方をインスタンス化して各々のメソッドを呼び出す。(calss methodは当然だがインスタンス化していない)

ちなみに電話とスマホを題材にしたのは、記事を書くときに手元にiphoneがあったから (笑)

細かい処理などはコード内にメモした。


## まずはJavaから
### Phoneクラス

```java:Phone.java
package com.company;

/**
 * 電話クラス
 */
public class Phone {
    /**
     * 電話番号
     */
    private String number;

    /**
     * コンストラクタ
     * @param number
     */
    public Phone(String number) {
        this.number = number;
    }

    /**
     * 自分の電話番号を表示する
     */
    public void showMyNumber(){
        System.out.println("This phone's number is " + this.number);
    }

    /**
     * 指定された相手に電話をかける
     * @param phonePartner 電話の相手
     */
    public void call(String phonePartner) {
        System.out.println("Call to " + phonePartner);
    }
}
```

### Smart Phoneクラス
Phoneクラスの子クラス

```java:SmartPhone.java
package com.company;

/**
 * スマホ
 * Phone classの子クラス
 */
public class SmartPhone extends Phone {
    /**
     * os
     */
    private String os;


    /**
     * コンストラクタ
     *
     * @param number 電話番号
     * @param os     os
     */
    public SmartPhone(String number, String os) {
        super(number);
        this.os = os;
    }


    /**
     * 指定された相手にTV電話をかける
     * オーバーライドした
     *
     * @param phonePartner 電話の相手
     */
    @Override
    public void call(String phonePartner) {
        System.out.println("Video call to" + phonePartner);
    }

    /**
     * OSを表示する
     */
    public void showMyOS() {
        System.out.println("his phone's os is" + this.os);
    }

    /**
     * クラスメソッド
     * 写真を取る
     *
     * @param subject 被写体
     */
    public static void takeAPitcure(String subject) {
        System.out.println("Take a picture of " + subject);
    }

}
```

### メインクラス
ここで各々のクラスのインスタンス化を行う。

```java:Main.java
package com.company;

/**
 * メイン
 */
public class Main {
    public static void main (String[] args) {
        System.out.println("===Normal Phone===");
        // インスタンス化
        Phone normalPhone = new Phone("xxx-xxx-xxx");
        normalPhone.showMyNumber();
        normalPhone.call("sekky0905");

        System.out.println("===Smart Phone===");
        // インスタンス化
        SmartPhone iPhone = new SmartPhone("○○○-○○○-○○○", "ios");
        iPhone.showMyNumber();
        iPhone.call("sekky0905");
        iPhone.showMyOS();
        SmartPhone.takeAPitcure("flower");

        System.out.println("===Smart Phone2===");
        // インスタンス化
        Phone zenPhone = new SmartPhone("△△△-△△△-△△△", "android");
        zenPhone.showMyNumber();
        zenPhone.call("sekky0905");
        // ちなみに当然だけどこれはできない
//        zenPhone.showMyOS();

    }

}
```


### Javaの実行結果
```
===Normal Phone===
This phone's number is xxx-xxx-xxx
Call to sekky0905
===Smart Phone===
This phone's number is ○○○-○○○-○○○
Video call tosekky0905
his phone's os isios
Take a picture of flower
===Smart Phone2===
This phone's number is △△△-△△△-△△△
Video call tosekky0905
```

## 続いてPythonのコード

```py3:main.py
# クラス
class Phone:
    # コンストラクタ
    def __init__(self, number):
        self.__number = number

    # Pythonのメソッドは必ず1つの引数を持つらしい
    # 第一引数は、必ずselfにするのが慣習らしい
    def show_my_number(self):
        print('This phone\'s number is{0}.'.format(self.__number))

    # Pythonのメソッドは必ず1つの引数を持つらしい
    # 第一引数は、必ずselfにするのが慣習らしい
    def call(self, phone_partner):
        print('Call to {0}.'.format(phone_partner))


# Phoneの子クラス
class SmartPhone(Phone):
    def __init__(self, number, os):
        super().__init__(number)
        self.os = os

    # Overrideした
    def call(self, phone_partner):
        print('Video call to {0}.'.format(phone_partner))

    def show_my_os(self):
        print('This phone\'s os is {0}.'.format(self.os))

    # クラスメソッド
    @classmethod
    def take_a_picture(cls, subject):
        print('Take a picture of {0}.'.format(subject))

print("===Normal Phone===")
# インスタンス化
normalPhone = Phone("xxx-xxx-xxx")
normalPhone.show_my_number()
normalPhone.call("sekky0905")

print("===Smart Phone===")
# インスタンス化
iPhone = SmartPhone("○○○-○○○-○○○", "ios")
iPhone.show_my_number()
# Phoneを継承しているから使える
iPhone.call("sekky0905")
iPhone.show_my_os()
# クラスメソッド
SmartPhone.take_a_picture("flower")
```

### Pythonの実行結果

```
===Normal Phone===
This phone's number isxxx-xxx-xxx.
Call to sekky0905.
===Smart Phone===
This phone's number is○○○-○○○-○○○.
Video call to sekky0905.
This phone's os is ios.
Take a picture of flower.
```

## 感想
Pythonは最初は抵抗があったけど、慣れてきて意外に書きやすいかもと思ってきた。
でもPython触った後に静的言語を書くと故郷に帰ってきた安心感がある。

新しい言語を学ぶのは楽しい~!

## 参考にさせていただいたサイト
[9. クラス — Python 3.6.1 ドキュメント](https://docs.python.jp/3/tutorial/classes.html#inheritance)

[Python基礎講座(13 クラス) - Qiita](http://qiita.com/Usek/items/a206b8e49c02f756d636)

※ ブログでも同一の記事を投稿している
[Python3とJavaでクラスを書いてみた -](http://www.sekky0905.com/entry/2017/07/25/Python3%E3%81%A8Java%E3%81%A7%E3%82%AF%E3%83%A9%E3%82%B9%E3%82%92%E6%9B%B8%E3%81%84%E3%81%A6%E3%81%BF%E3%81%9F)
