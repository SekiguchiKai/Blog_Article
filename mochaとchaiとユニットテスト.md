# mochaとchaiとユニットテスト

#この記事の内容
①テストとは、②mochaとchaiを使ったNode.jsテスト


#ソフトウェアテスト

>コンピュータのソフトウェアプログラムを実行し、それが意図したとおりに動くかを観測・評価・検証する作業のこと。通常は検証対象のソフトウェアを実際に試行する動的テストを指すが、レビューなどを静的テストと呼んで広義のテストに含める見方もある。

引用元
http://www.itmedia.co.jp/im/articles/1111/07/news192.html

#テストケースとは
>ソフトウェア開発におけるテストは一般に、ユーザーの使用状況を反映した入力データを用いてソフトウェアを実行し、事前に想定した結果と実際の実行結果をつき合わせて合否の判定を行う。このテスト用の“入力データ”とそれを実行したら得られるであろう“事前に想定した結果”の対をテストケースという。すなわち、テストケースを使って対象を実行してみる“行為”がテスト（testing）である。

引用元
http://www.itmedia.co.jp/im/articles/1111/07/news192.html


#単体テスト（ユニットテスト）
>プログラムを構成する最小単位で実行されるソフトウェアテストのこと。モジュールテストともいう。

引用元
http://www.itmedia.co.jp/im/articles/1111/07/news132.html

#テスティングフレームワーク
>　ソフトウェアテストの自動実行プログラムの作成、テスト結果の自動判定や自動集計などの機能を提供するソフトウェアフレームワークのこと。

引用元
http://www.itmedia.co.jp/im/articles/1111/07/news184.html

#テストで大切なこと（ここ最近学んだこと）
* 目的から考えること<br>
=>このテストは何をテストすることが目的なのか

* テスト対象の責任をなるべく小さくすること


#Mocha
* テスト（BDDやTDD）をするための枠組み
* テスティングフレームワーク

#Chai
テストをするのに必要なメソッド(アサーション)を提供


#テストの書き方

* テストコードは、testというディレクトリを作成し、その中に格納しなくてはならない
* テストを実行する際には、```mocha```とコマンドを打てば、テストが自動で走る

```js
// assertアサーションを利用できるようにする
var chai = require('chai');
var assert = chai.assert;
//
var hoge = require('ソースコードを書く');

//describe()は、複数のテストケースをまとめるためのもの
descrbe('このテストの題名', function(){

    it('テスト項目の期待する結果を説明する言葉', function(){
        //アサーションを用いて、ここのテストを実施(assert.メソッドという形で使われる)
        //このアサーションテスト結果と期待値を比較し、真偽値判定する
    });

    it('テスト項目の期待する結果を説明する言葉', function(){
        //アサーションを用いて、ここのテストを実施(assert.メソッドという形で使われる)
        //このアサーションテスト結果と期待値を比較し、真偽値判定する
    });

    it('テスト項目の期待する結果を説明する言葉', function(){
        //アサーションを用いて、ここのテストを実施(assert.メソッドという形で使われる)
        //このアサーションテスト結果と期待値を比較し、真偽値判定する
    });

});
```

#具体例

テストされる側のコード

```js
// じゃんけんのアルゴリズム部分（勝敗を決定してresultを返す）
/* c = クライアントの打ち手、 s = サーバの打ち手（デフォルト値）
   cは、クライアントから送られてくる、sは、デフォルト値を使うこととし、テストの場合のみ変数を与える
*/


const Uchite_GU = 0;
const Uchite_TYOKI = 1;
const Uchite_PA = 2;


// クライアントとサーバのじゃんけんの結果を判定する関数 // 乱数を別に　この関数から呼び出す
exports.judgeResult = function judgeResult(c, s = Math.floor(Math.random() * (2 - 0 + 1)) + 0) {

    // 確認用
    console.log("janken.jsrが呼び出されました。レスポンスのための資源を生成します");

    // クライアントの打ち手に引数を代入
    var clientUchite = c;

    // サーバの打ち手を代入に引数を代入
    var serverUchite = s;

    // じゃんけんの結果を格納する変数を宣言
    var result;

    // アルゴリズム
    if ((clientUchite === Uchite_GU && serverUchite === Uchite_TYOKI) || (clientUchite === Uchite_TYOKI && serverUchite === Uchite_PA) || (clientUchite === Uchite_PA && serverUchite === Uchite_GU)) {
        result = "君の勝ちだ！";
    }
    else if ((clientUchite === Uchite_TYOKI && serverUchite === Uchite_GU) || (clientUchite === Uchite_PA && serverUchite === Uchite_TYOKI) || (clientUchite === Uchite_GU && serverUchite === Uchite_PA)) {
        result = "君の負けだ！";
    }
    else if (clientUchite === serverUchite) {
        result = "引き分けだ！";
    }

    // 表示のための処理
                // クライアント
                switch (clientUchite) {
                    case Uchite_GU:
                        clientUchite = "グー"
                        break;
                    case Uchite_TYOKI:
                        clientUchite = "チョキ"
                        break;
                    case Uchite_PA:
                        clientUchite = "パー"
                        break;
                }

                // 表示のための処理
                // サーバ
                switch (serverUchite) {
                    case Uchite_GU:
                        serverUchite = "グー"
                        break;
                    case Uchite_TYOKI:
                        serverUchite = "チョキ"
                        break;
                    case Uchite_PA:
                        serverUchite = "パー"
                        break;
                }
    // オブジェクトとして各値を返す
    var allResultObj= {};
    // オブジェクトの各プロパティに値を追加
    allResultObj.result = result;
    allResultObj.clientUchite = clientUchite;
    allResultObj.serverUchite = serverUchite;
    return allResultObj;
}
```

テストする側のコード

```js
// webServer.jsのテストをBDDで行う

// assertアサーションを利用できるようにする
var chai = require('chai');
var assert = chai.assert;

// webServer.jsをmoduleとして使用するために
var jk = require('../src/janken.js');

const UCHITE_GU = 0;
const UCHITE_TYOKI = 1;
const UCHITE_PA = 2;


// judgeResultの戻り値のオブジェクトの中から、resultのみを取り出す関数
function objConverter(client, sever) {
    var objFolder;
    objFolder = jk.judgeResult(client, sever).result;
    return objFolder;
}


// describe()は、複数のテストケースをまとめるためのもの
describe('じゃんけんアルゴリズム', function () {
    // 事前に行う処理

    it('「クライアント:グー、サーバ:チョキ」の場合は、クライアントの勝ち', function () {
        assert.equal(objConverter(UCHITE_GU, UCHITE_TYOKI), "君の勝ちだ！");
    });

    it('クライアント:チョキ、サーバ:パー」の場合は、クライアントの勝ち', function () {
        assert.equal(objConverter(UCHITE_TYOKI, UCHITE_PA), "君の勝ちだ！");
    });

    it('クライアント:パー、サーバ:グー」の場合は、クライアントの勝ち', function () {
        assert.equal(objConverter(UCHITE_PA, UCHITE_GU), "君の勝ちだ！");
    });

    it('クライアント:チョキ、サーバ:グー」の場合は、サーバの勝ち', function () {
        assert.equal(objConverter(UCHITE_TYOKI, UCHITE_GU), "君の負けだ！");
    });

    it('クライアント:パー、サーバ:チョキ」の場合は、サーバの勝ち', function () {
        assert.equal(objConverter(UCHITE_PA, UCHITE_TYOKI), "君の負けだ！");
    });

    it('クライアント:グー、サーバ:パー」の場合は、サーバの勝ち', function () {
        assert.equal(objConverter(UCHITE_GU, UCHITE_PA), "君の負けだ！");
    });

    it('クライアント:グー、サーバ:グー」の場合は、引き分け', function () {
        assert.equal(objConverter(UCHITE_GU, UCHITE_GU), "引き分けだ！");
    });

    it('クライアント:チョキ、サーバ:チョキ」の場合は、引き分け', function () {
        assert.equal(objConverter(UCHITE_TYOKI, UCHITE_TYOKI), "引き分けだ！");
    });

    it('クライアント:パー、サーバ:パー」の場合は、引き分け', function () {
        assert.equal(objConverter(UCHITE_PA, UCHITE_PA), "引き分けだ！");
    });

    it('クライアントをグー固定で1000回対戦してみて、勝ち・負け・引き分け のそれぞれが最低1回以上発生するかというテスト', function () {
        // クロームでのデバッグのために、timeoutの期限を伸ばす
        this.timeout(999999999999);
        // var resultArray = new Array(1000);
        var resultArray = [];
        for (var i = 0; i < 1001; i++) {
            // クライアントの引数をグーで固定し、janken()を1000回呼び出し
            resultArray.push(objConverter(UCHITE_GU));
        }
        // テストで、janken.jsのjdugeResult関数に処理させて返ってきた結果が正しいかどうかを判断するための関数を格納したオブジェクト
        var judgeAdequacy = {
            returnAdequency: function (result) {
                // judgeResult関数を使用して、テストした勝負の結果を格納する配列
                var resultArray = result;
                // テスト結果が適切かどうかを判断した結果を格納する変数
                var resultAdequacy = "";

                // janken.jsのjdugeResult関数に1000回処理させた結果を格納したresultArrayにテストとして正しい値が格納されているか判断
                if ((resultArray.indexOf("君の勝ちだ！") >= 0) && (resultArray.indexOf("君の負けだ！") >= 0) && (resultArray.indexOf("引き分けだ！") >= 0)) {
                    resultAdequacy = "合格";
                } else {
                    resultAdequacy = "不合格";
                }
                return resultAdequacy;
            }

        }
        assert.equal(judgeAdequacy.returnAdequency(resultArray), "合格");
    });

});
```



参考にさせていただいたサイト
http://qiita.com/y_hokkey/items/f73ea6b3d5f6902396b6
http://phiary.me/javascript-test-framework-mocha-usage/
http://numb86-tech.hatenablog.com/entry/2016/06/08/155834
http://hokaccha.hatenablog.com/entry/20111202/1322840375
