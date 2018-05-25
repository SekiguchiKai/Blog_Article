# Angular4(over2~)のロケール


## ローケルとは
> ロケールとは、ソフトウェアに内蔵される、言語や国・地域ごとに異なる単位、記号、日付、通貨などの表記規則の集合。または単に、利用する言語や国・地域の指定。多くのソフトウェアやプログラミング言語は、使用する言語とともにロケールを設定し、ロケールで定められた方式に基づいてデータの表記や処理を行う。

引用元 : [ロケール（ロカール）とは - IT用語辞典](http://e-words.jp/w/%E3%83%AD%E3%82%B1%E3%83%BC%E3%83%AB.html)

つまり、各国・地域などによって異なる諸々の表記方法であり、今回はこれを設定する。

## ローケルとPipe
Angularの[Pipe](https://angular.io/guide/pipes)では、このローケルの設定によって、処理結果が異なるものが多々存在する。

## DatePipe
[DatePipe](https://angular.io/api/common/DatePipe)もローケルの設定によって、処理結果が変わるPipeの1つである。
例えば、ロケールを設定しないで、以下のコードを実行する。

```js:app.component.ts
import {Component} from '@angular/core';

@Component({
  selector: 'app-root',
  template: `<p class="today">
本日は、{{today | date}} だ
</p>
`,
  styles: [`
.today{background-color:yellow}
`]
})
export class AppComponent {
  today = Date.now();
}
```

すると、表示される画面は以下のようになる。
![スクリーンショット 2017-08-30 13.01.34.png](https://qiita-image-store.s3.amazonaws.com/0/145611/ce49e744-bb57-8c62-8dc8-e035b6a9420c.png)



### Angularのアプリケーションにロケールを設定する
[LOCALE_ID](https://angular.io/api/core/LOCALE_ID)を使用し、アプリケーションにロケールを設定する。
LOCALE_IDをprovideする際にuseValueでLocaleIdを与えると、アプリケーションのロケールがLocaleIdで渡されたものに設定される

### 実際にやってみた
今回は、ロケールを日本に設定してみる。
app.module.tsのproviderでLOCALE_IDをprovideする際にuseValueで与えるLocaleIdを `js-JP` (日本)にする。
実際のコードは以下。

```js:app.module.ts
import {BrowserModule} from '@angular/platform-browser';
import {NgModule, LOCALE_ID} from '@angular/core';
import {AppComponent} from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule],
  providers: [{provide: LOCALE_ID, useValue: 'ja-JP'}],
  bootstrap: [AppComponent]
})
export class AppModule {
}
```

componentは先ほどと変わらず以下のようなものにする。

```js:app.component.ts
import {Component} from '@angular/core';

@Component({
  selector: 'app-root',
  template: `<p class="today">
本日は、{{today | date}} だ
</p>
`,
  styles: [`
.today{background-color:yellow}
`]
})
export class AppComponent {
  today = Date.now();
}
```

#### 実行結果
実行すると以下のようになり、ロケールが日本になったため、同じコードでもDataPipeで変換された後の値が変化していることがわかる。

![スクリーンショット 2017-08-30 13.16.31.png](https://qiita-image-store.s3.amazonaws.com/0/145611/14b61ce0-65d2-f8a6-cd63-8135599e0aa2.png)


## 参考文献
山田 祥寛(2017/8/4)『Angularアプリケーションプログラミング』 技術評論社

## 参考にさせていただいたサイト
 [ロケール（ロカール）とは - IT用語辞典](http://e-words.jp/w/%E3%83%AD%E3%82%B1%E3%83%BC%E3%83%AB.html)
[Angular - Pipes](https://angular.io/guide/pipes)
[DatePipe](https://angular.io/api/common/DatePipe)
[LOCALE_ID](https://angular.io/api/core/LOCALE_ID)

※ ブログでも同一の投稿を行っている
[Angular4(over2~)のロケールとPipe](http://sekky0905.hatenablog.com/entry/2017/08/31/Angular4%28over2~%29%E3%81%AE%E3%83%AD%E3%82%B1%E3%83%BC%E3%83%AB%E3%81%A8Pipe)

