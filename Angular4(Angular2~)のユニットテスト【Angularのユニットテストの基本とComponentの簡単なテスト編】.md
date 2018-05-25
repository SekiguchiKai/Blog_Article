# Angular4(Angular2~)のユニットテスト【Angularのユニットテストの基本とComponentの簡単なテスト編】

Angularのユニットテストを勉強中なので、何回かに分けて記事にする。
今回の記事は、以下の公式ドキュメントの4つのセクションをかなり参考にさせていただいている。
[Introduction to Angular testing](https://angular.io/docs/ts/latest/guide/testing.html#!%23testing-intro)
[The first karma test](https://angular.io/docs/ts/latest/guide/testing.html#!%231st-karma-testo)
[Test a component](https://angular.io/docs/ts/latest/guide/testing.html#!%23simple-component-test)
[Test a component with an external template](https://angular.io/docs/ts/latest/guide/testing.html#!%23component-with-external-template)

今回は、Angularのテストの大まかな流れと、簡単なComponentのテストを記事の対象とする。

## 使うテストツール
Angularのテストでは、JavaScript界のいくつかのテストツールを使用する。それらのツールを理解していないと挫折するので、まずは、Angular自体のテストに先立って、それらのツールを学ぶ必要がある。以下を参考にするとわかりやすい。

### jasmine
JavaScript 向けテスティングフレームワーク

jasmine公式
https://jasmine.github.io/2.0/introduction.html
jasmineのわかりやすい記事
[Jasmine使い方メモ - Qiita](http://qiita.com/opengl-8080/items/cf3acafda9756f4b04c9)
[Jasmine spec覚え書き - Qiita](http://qiita.com/hmsk/items/8f6965968692186b1ea1)

### karma
ブラウザでユニットテストを実行するためのテストランナー

karmaのわかりやすい記事
[step by stepで始めるKarma - Qiita](http://qiita.com/howdy39/items/b9d704e7f84053924da3)
karma公式
[Karma - Spectacular Test Runner for Javascript](https://karma-runner.github.io/)

## 被テストComponent

``` ts:app.component.ts
import {Component} from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'テストだよ';

  /**
   * h1の文章を変更する
   */
  changeH1Element() {
    this.title = 'クリックされたぜ!';
  }
}
```

``` html:app.component.html
<h1>
  {{title}}
</h1>

<p class="button" (click)="changeH1Element()">ここをクリックして</p>
```




タイトルがあって、「ここをクリックして」というところを押すと、タイトルが変更される。
なお、今回の目的はAngularのユニットテストの学習のため、CSSなどによる装飾等は一切行わない。


## テストクラス

``` ts:app.component.spec.ts
import {ComponentFixture, TestBed, async, ComponentFixtureAutoDetect,} from '@angular/core/testing';
import {By}              from '@angular/platform-browser';
import {DebugElement}    from '@angular/core';
import {AppComponent} from './app.component';


// describeでテストSuiteを作成
describe('AppComponentのテスト', () => {
  // テストの中のAppComponent
  let comp: AppComponent;
  // ComponentFixtureは、 componentのインスタンスそのものとcomponentのDOM elementのハンドルであるDebugElementへのアクセスを提供する。
  let fixture: ComponentFixture<AppComponent>;
  // ComponentのDOM elementのhandle
  let de: DebugElement;
  let el: HTMLElement;


  // 各Spec(個々のテスト)が開始される前に行う処理を設定する。
  // 非同期処理
  // Componentのインスタンスを生成する前に、Angular template compilerが外部ファイルを読み込む
  beforeEach(async(() => {
    // テストしたいクラスのためのモジュール環境をconfigureTestingModuleメソッドで設定する。
    // メタデータの登録
    TestBed.configureTestingModule({
      // テストされるComponentを登録
      declarations: [
        AppComponent
      ],
      providers: [
        {provide: ComponentFixtureAutoDetect, useValue: true}
      ]
    }).compileComponents(); // 外部ファイルをコンパイル

  }));

  // 同期処理
  // Componentのインスタンスを生成
  beforeEach(() => {
    // ComponentFixtureは、 componentのインスタンスそのものとcomponentのDOM elementのハンドルであるDebugElementへのアクセスを提供する。
    fixture = TestBed.createComponent(AppComponent);

    // テストされるComponentのインスタンス
    comp = fixture.componentInstance;

    // queryは、fixtureのDOM全体の中から、引数で与えたelementを満たす最初のDom elementとマッチしたDebugElementを返す
    // Byでは、cssのselectorを生成している。
    de = fixture.debugElement.query(By.css('h1'));

    el = de.nativeElement;
  });


// itでSpecを作成
  it('AppComponentのインスタンスが生成できているかどうか', async(() => {
    // trueかどうかs
    expect(comp).toBeTruthy();
  }));
  it('何もしない場合のタイトルがAppComponentのtitleと同じかどうか', async(() => {
    expect(el.textContent).toContain(comp.title);
  }));
  it('何もしない場合のタイトルがAppComponentのtitleと同じかどうか(上のテストと同じことをしている)', async(() => {
    expect(el.textContent).toContain('テストだよ');
  }));

  it('detectChangesが1回起きた時、h1の値が変更されるかどうか', async(() => {
    comp.title = 'クリックされたぜ!';
    fixture.detectChanges();
    expect(el.textContent).toContain(comp.title);
  }));

  it('changeH1Elementが呼び出されたら、titleが変更されるかどうか', async(()=> {
    comp.changeH1Element();
    expect(comp.title).toBe('クリックされたぜ!');
  }));
});

```


各々の処理で行っていることは、コード中にコメントとして記述してある。

## Angularのテストの大まかな構成
基本的には、jasmineの構成に則っているようである。
1. beforeEachで、各テストSpecの処理の前に行う処理を記述する。(場合によってはafterEachで、各テストSpecの処理の後に行う処理を記述する。)
2. descibeでSuite(テストの塊のようなもの)を作成し、その中でitでSpec(個々のテストメソッド)を複数記述する。このitの中で、様々なテストを行う。

## Angularのテストで使用するクラスとメソッド
### TestBed
公式APIリファレンス
https://angular.io/docs/ts/latest/api/core/testing/index/TestBed-class.html

> Configures and initializes environment for unit testing and provides methods for creating components and services in unit tests.
引用元:https://angular.io/docs/ts/latest/api/core/testing/index/TestBed-class.html

(意訳)
ユニットテスト用の設定と環境の初期化を行う。
また、ユニットテスト内のcomponentとserviceを生成するためのメソッドを作成する。

> It creates an Angular testing module—an @NgModule class—that you configure with the configureTestingModule method to produce the module environment for the class you want to test.
引用元: https://angular.io/docs/ts/latest/guide/testing.html#!%23q-spec-file-location

(意訳)
TestBedは、Angularのテスティングモジュール([@NgModule](https://angular.io/docs/ts/latest/api/core/index/NgModule-interface.html)クラス)を生成する。
[@NgModule](https://angular.io/docs/ts/latest/api/core/index/NgModule-interface.html)クラスはテストしたいクラスのためのモジュール環境をconfigureTestingModuleメソッドで設定する。

=> テストの環境初期化と設定を行う。

### TestBed.configureTestingModule
テストしたいクラスのためのモジュール環境を設定する。
メタデータの登録を行う。[@NgModule](https://angular.io/docs/ts/latest/api/core/index/NgModule-interface.html)のテスト版のようなもの

### beforeEach (jasmineのメソッド)
各Spec(個々のテスト)が開始される前に行う処理を設定する。

### TestBed.createComponentとComponentFixture
TestBed.createComponent　公式APIリファレンス
https://angular.io/docs/js/latest/api/core/testing/index/TestBed-class.html
ComponentFixture　公式APIリファレンス
https://angular.io/docs/ts/latest/api/core/testing/index/ComponentFixture-class.html

> The createComponent method returns a ComponentFixture, a handle on the test environment surrounding the created component. The fixture provides access to the component instance itself and to the DebugElement, which is a handle on the component's DOM element.
引用元: https://angular.io/docs/ts/latest/guide/testing.html#!%23component-fixture

(意訳)
createComponent(TestBed.createComponent)は、生成されたcomponentのテスト環境のハンドルであるComponentFixtureクラスを返す。ComponentFixtureは、 componentのインスタンスそのものとcomponentのDOM elementのハンドルであるDebugElementへのアクセスを提供する。

⇒ ComponentFixture = テストしたいComponentのインスタンスとそのComponentのDebugElementへのアクセスを提供するもの。
しつこいようだが、これによってテスト対象のComponentのインスタンスとDOM elementを操作できるようになる。

TestBed.createComponentは、TestBedインスタンスの設定を閉じるので、createComponentを使用した後にTestBedの設定系のメソッドを使用してはいけないようである。


### DebugElement.query
公式APIリファレンス
https://angular.io/docs/ts/latest/api/core/index/DebugElement-class.html

queryは、fixtureのDOM全体の中から、引数で与えたelementを満たす最初のDom elementとマッチしたDebugElementを返す。fixture.debugElement.queryのDebugElementと戻り値のDebugElementは異なる。

queryAllの場合は、引数で与えたelementを満たす全てのDom elementの配列を返す。

参考
https://angular.io/docs/ts/latest/guide/testing.html#!%23component-fixture
https://angular.io/docs/js/latest/api/core/index/DebugElement-class.html#!%23query-anchor

### By
AngularのUtilityで、 predicates(述語と単純に訳していいのか不明)を生成する。

公式APIリファレンス
https://angular.io/docs/ts/latest/api/platform-browser/index/By-class.html

### detectChanges
componentのchange detection cycleを行う。

公式APIリファレンス
https://angular.io/docs/ts/latest/api/core/testing/index/ComponentFixture-class.html#!%23detectChanges-anchor
detectChangesについては、以下がわかりやすい。
[日本語訳：Angular 2 Change Detection Explained - Qiita](http://qiita.com/laco0416/items/523d96ddbfe55c4e6949)

以下の記事の言葉をお借りすれば、
> change Detectionとはモデルの変更を検知し、UIに反映することである

とのこと
引用元 [日本語訳：Angular 2 Change Detection Explained - Qiita](http://qiita.com/laco0416/items/523d96ddbfe55c4e6949)

fixture.detectChanges()を呼ぶことで、テストからAngilarにchange detection が行われたことを伝える。

## Angularで外部のtemplateを読み込む
外部のtemplateやcssファイルを使用するときは、ちょっとした処理が必要なようだ。

> But the Angular template compiler must read the external files from the file system before it can create a component instance. That's an asynchronous activity.
引用元: https://angular.io/docs/ts/latest/guide/testing.html#!%23component-fixture

(意訳)
Angularでは、Componentのインスタンスを生成する前に、Angular template compilerが外部ファイルを読み込む必要があり、これは非同期な活動である。

⇒
つまり、Componentのインスタンスを生成する前にAngular template compilerが外部ファイルを読み込めるような処理をbeforeEachの中で行う必要があるわけである。


### TestBed.compileComponents
公式APIリファレンス

まず、テスティングモジュールで設定されたComponentを非同期でコンパイルする。
上記のcompileComponentsが完了したら、外部のファイルや、cssファイルはinlinedされていて、
TestBed.createComponentは、同期的にComponentのインスタンスを生成することができる。

参考: https://angular.io/docs/ts/latest/guide/testing.html#!%23component-fixture

templateやcssを外部ファイルにするときは、必ず「Componentのインスタンスを生成する」前に「外部ファイルを読み込む」ように非同期、同期をうまく使う。(コードの記述はこの順番でなくても構わない)

## テスト実行画面
以下のコマンドをコマンドライン上で打つとテストランナーであるkarmaによってブラウザが立ち上がり、テスト結果がブラウザ上に表示される。

```
ng test
```

<img width="794" alt="ng-karma.png" src="https://qiita-image-store.s3.amazonaws.com/0/145611/2c3dd122-4da0-9287-8159-9fd95fcba82c.png">


## 参考にさせていただいたサイト
公式
https://angular.io/docs/ts/latest/guide/testing.html#!%23component-fixture

jasmine公式
https://jasmine.github.io/2.0/introduction.html
jasmineわかりやすい記事
[Jasmine使い方メモ - Qiita](http://qiita.com/opengl-8080/items/cf3acafda9756f4b04c9)
[Jasmine spec覚え書き - Qiita](http://qiita.com/hmsk/items/8f6965968692186b1ea1)

karmaのわかりやすい記事
[step by stepで始めるKarma - Qiita](http://qiita.com/howdy39/items/b9d704e7f84053924da3)
karma公式
[Karma - Spectacular Test Runner for Javascript](https://karma-runner.github.io)

公式APIリファレンス
https://angular.io/docs/ts/latest/api/core/testing/index/TestBed-class.html
https://angular.io/docs/js/latest/api/core/testing/index/TestBed-class.html
https://angular.io/docs/ts/latest/api/core/testing/index/ComponentFixture-class.html
https://angular.io/docs/ts/latest/api/core/index/DebugElement-class.html
https://angular.io/docs/ts/latest/api/platform-browser/index/By-class.html
https://angular.io/docs/ts/latest/api/core/testing/index/ComponentFixture-class.html#!%23detectChanges-anchor


detectChangesについて
[日本語訳：Angular 2 Change Detection Explained - Qiita](http://qiita.com/laco0416/items/523d96ddbfe55c4e6949)

---

※ ブログでも同一の記事を投稿している。

[Angular4(Angular2~)のユニットテスト【Angularのユニットテストの基本とComponentの簡単なテスト編】](http://www.sekky0905.com/entry/2017/06/08/Angular4%28Angular2~%29%E3%81%AE%E3%83%A6%E3%83%8B%E3%83%83%E3%83%88%E3%83%86%E3%82%B9%E3%83%88%E3%80%90Angular%E3%81%AE%E3%83%A6%E3%83%8B%E3%83%83%E3%83%88%E3%83%86%E3%82%B9%E3%83%88%E3%81%AE%E5%9F%BA)
