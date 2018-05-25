# RxJSの基本をまとめてみた~基本的な概念編~
Angularで使用していたRxJSだが、もっと深くちゃんと理解しようと思い学習したので、これから何回かに分けてまとめる。
仕組みを理解するために、ちょっと冗長な書き方をするところがある。
「これどうなの?」とか「意訳おかしくね?」という部分があったら、ご指摘いただけるとありがたいです。

## 今回の範囲
今回は、RxJSやリアクティブプログラミングの基本的な概念である、データストリーム、Observable、Observer、Subscription関連についてまとめたいと思う。
今回は、[RxJSの公式ドキュメント](http://reactivex.io/rxjs/manual/overview.html#subscription)をかなり参考にさせていただいた。

##RxJS
### RxJSとは
JavaScript向けのReactive Extensions ライブラリで、リアクティブプログラミングを行うためのもの。
[ReactiveX/rxjs: A reactive programming library for JavaScript](https://github.com/ReactiveX/rxjs)

## リアクティブプログラミング
### リアクティブプログラミングとは
リアクティブプログラミングについて様々な説明のされ方があるが、以下の説明が一番ピンときた。

> リアクティブプログラミングとは、通知されてくるデータを受け取るたびに関連したプログラムが反応し(リアクション)して、処理を行うようにするプログラミングの考え方です。

引用元 : 須田智之 (2017/2/16)『RxJavaリアクティブプログラミング』 翔泳社

### データストリーム
リアクティブプログラミングでは、あらかじめ用意されている固定長のデータだけではなく、随時発生するデータを処理することができる。
この随時データが生成されて、その都度流れてくるデータの流れをデータストリームという。

 参考 :
[「RxJS」初心者入門 - JavaScriptの非同期処理の常識を変えるライブラリ](https://liginc.co.jp/web/js/151272) 
須田智之 (2017/2/16)『RxJavaリアクティブプログラミング』 翔泳社

## RxJSの基礎
### ProducerとConsumer
#### Producer 
データを生成し、そのデータを通知する責務を持つ。
RxJSではObservableがこれに当たる。

>  An Observable emits items or sends notifications to its observers by calling the observers’ methods.

引用元 : [ReactiveX - Observable](http://reactivex.io/documentation/observable.html)

【意訳】Observableは、アイテム(データなど)を排出し、observerのメソッドを呼び出すことによって、自身のobserverに通知を送信する。

#### Consumer
データを受信し、必要な処理を行う責任を持つ。
RxJSではObserverがこれに当たる。

> In ReactiveX an observer subscribes to an Observable. Then that observer reacts to whatever item or sequence of items the Observable emits.

引用元 : [ReactiveX - Observable](http://reactivex.io/documentation/observable.html)

【意訳】ReactiveXでは、ObserverはObservableを購読する。さらに、ObserverはObservableが生成するいかなるアイテム(データなど)やアイテム(データなど)のシーケンスに対して反応する。

### ObservableとObserverとオペレータとsubscribe
Observableがデータを生成し、データを通知する。

Observableがデータを生成し、Observerにデータを通知し、届けるまでに、様々な前処理を行うことができる。
この前処理をオペレータという。
オペレータを使って前処理をすると新しいObservableを生成することになる。
オペレータに関しては別で記事を書きたいと思っている。

そして、Observableシーケンス(データストリーム)から流れてくるデータをObserverがsubscribe(購読)する。その際に、受け取ったデータに関して、必要な処理を行う。

上記の処理の流れをエラーが出るか、完了するまで続ける。
エラーや完了については後述する。


ただ、実際のコードの中だと「Observerってどこで生成されているの!?」と思う　ことが多々ある。
それは以下のような理由らしい。

> Internally in observable.subscribe, it will create an Observer object using the first callback argument as the next handler. All three types of callbacks may be provided as arguments

引用元 : [RxJS Overview](http://reactivex.io/rxjs/manual/overview.html#subscribing-to-observables

【意訳】
`observable.subscribe` で内部的に、最初のコールバックの引数をnextハンドラーとして使用しているObserverのオブジェクトを生成する。3つのコールバックの全部のタイプが、引数として提供され得る。


#### Observerの3つのコールバック
> Observers are just objects with three callbacks, one for each type of notification that an Observable may deliver.

引用元 : http://reactivex.io/rxjs/manual/overview.html#subscribing-to-observables

【意訳】
Observerは、Observableが届けてくるであろう各々のタイプの通知に対する3つのコールバックを持ったただのオブジェクトである。

Observerは、Observableをsubscribe(購読)し、受け取ったデータを処理する。
ここでいう3つのコールバックがその処理に当たる。
3つのコールバックとは、 `next` 、 `error` 、`complete` である。

#### `next`
Observableがデータを生成する度にObservableによって呼び出される。
Observableによって生成されたデータを引数に取る。

#### `error` 
エラーが発生したことを通知する。
これが呼び出されると、 `next` や `complete` はそれ以上、呼び出されない。
引数には、エラーの原因が渡される。

#### `complete` 
完了したことを通知する。
エラーがなかった場合、最後のnextメソッドが呼び出された後に、Observableによって呼び出される。

参考: [ReactiveX - Observable](http://reactivex.io/documentation/observable.html)

### Subscription

> A Subscription is an object that represents a disposable resource, usually the execution of an Observable. A Subscription has one important method, unsubscribe, that takes no argument and just disposes the resource held by the subscription. 

引用元 : [RxJS Overview](http://reactivex.io/rxjs/manual/overview.html#subscription)

【意訳】
Subscriptionとは、破棄可能なリソースを表し、それは大抵Observableの実行を表すオブジェクトである。
Subscriptionは、引数を取らず、subscriptionが持っているリソースの破棄を行うだけのunsubscribeという重要なメソッドを1つ持っている。

=> データストリームであるObservableシケーンスが終わりがなく、随時生成されるデータだった場合に、そのデータストリームの実行状態を終わらせるのにSubscription#unsubscribeを使用するというわけである。

subscribeを呼び出した時にその戻り値として、Subscriptionが返却される。
以下のような感じで使う。

```ts


// observable.subscribeの戻り値として、subscriptionを返却
// observable.subscribeの内部的に、Observerのオブジェクトが生成されていている。
let subscription = observable.subscribe(val => console.log(val));
// 進行中の実行をキャンセル
subscription.unsubscribe();
```

### 実際にコードを書いてみた
実際に超基礎的なコードを書いてみた。
これまでにまとめてきた概念などを理解しやすくするために、かなり簡単なコードになっているし、普通はもっと合理的な書き方をした方が良い。
それぞれの処理が何をやっているかなどの細かい説明は、コードの中にコメントとして記述してある。

#### 実際のコード
typescript2.5.2を使用している。

```ts

import {Observable} from "rxjs";

// Observable.createは、ObservableのコンストラクタのエイリアスでObservableのオブジェクトを生成する
// 引数に、subscribe関数を渡す
// Observableはcreateで作れるけど、基本的にcreateを使って、Observableを生成することはあまりない
// ofとかの別のoperatorから生成する
let observable = Observable.create(function subscribe(observer) {
    try { // 通常時
        // データを通知し、送信する
        // 引数は、実際にObserverに送信されるデータを表している。
        observer.next(0);
        observer.next(1);
        observer.next(2);
        observer.next(3);
        observer.next(4);
        observer.next(5);

        // 正常に完了したことを通知する
        // これが呼ばれた後にnext()しても意味がない
        observer.complete();
    } catch (err) { // 異常時
        // エラーを通知する
        observer.error(err);
    }
});
console.log('開始');
// observable.subscribeの内部的に、Observerのオブジェクトが生成されている
observable.subscribe(val => console.log(val));


console.log('終了');
```

#### 実行結果
```
開始
0
1
2
3
4
5
終了
```
## 参考
### 参考文献
 須田智之 (2017/2/16)『RxJavaリアクティブプログラミング』 翔泳社

### 参考にさせていただいたサイト
[ReactiveX/rxjs: A reactive programming library for JavaScript](https://github.com/reactivex/rxjs)

[RxJS API Document](http://reactivex.io/rxjs/)
(http://reactivex.io/documentation/observable.html))
[ReactiveX - Observable](http://reactivex.io/documentation/observable.html)
[「RxJS」初心者入門 - JavaScriptの非同期処理の常識を変えるライブラリ](https://liginc.co.jp/web/js/151272)
http://reactivex.io/rxjs/manual/overview.html#subscribing-to-observables

※ [ブログ](http://sekky0905.hatenablog.com/entry/2017/09/19/RxJS%E3%81%AE%E5%9F%BA%E6%9C%AC%E3%82%92%E3%81%BE%E3%81%A8%E3%82%81%E3%81%A6%E3%81%BF%E3%81%9F~%E5%9F%BA%E6%9C%AC%E7%9A%84%E3%81%AA%E6%A6%82%E5%BF%B5%E7%B7%A8%28Observable%E3%80%81Observer%E3%80%81Sub)でも同一の投稿を行っている。
