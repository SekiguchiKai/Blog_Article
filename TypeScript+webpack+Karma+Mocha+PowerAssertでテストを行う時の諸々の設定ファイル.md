# TypeScript + webpack + Karma + Mocha + Power Assertでテストを行う時の諸々の設定ファイル

最近、TypeScriptでコードを書いて、モジュールバンドラにwebpack、テストランナーにkarma、テスティングフレームワークにmocha、アサーションライブラリにPower Assertを使用したユニットテストを書いた

その際に、諸々のツールが複雑で、設定ファイルの記述も大変だったので、ここで設定ファイルについてのメモを残しておこうと思う

※現段階では、設定ファイル自体にメモを書いただけです、後で個々の説明を書くかもしれません

## package.json
まずはこれがないと始まらないですよね

``` 
{
  // パッケージの名前
  "name": "パッケージ名",
  // パッケージのバージョン
  "version": "1.0.0",
  // パッケージの説明
  "description": "パッケージの説明文",
  // 項目はパッケージの中で最初に呼ばれるモジュールのID
  "main": "./dist/hoge.js",
  // scriptコマンド
  // key : イベント、value : コマンド
  // keyで登録したイベントをnpmで行うときにvalueのコマンドが実行される
  "scripts": {
    // npm startで実行される
    "start": "open resources/index.html",
    // ビルド:実行可能ファイルを作成
    "build": "webpack",
    // テスト実行前に行うコマンド
    "pretest": "npm run build",
    // テストを実行
    "test": "karma start"
  },
  // 作者
  "author": "sekky0905",
  // ライセンス
  "license": "MIT",
  "devDependencies": {
    // DefinitelyTypedの型定義ファイル集積リポジトリからmochaの型定義ファイルをインストール
    "@types/mocha": "^2.2.39",
    // DefinitelyTypedの型定義ファイル集積リポジトリからpower-assertの型定義ファイルをインストール
    "@types/power-assert": "^1.4.29",
    // ブラウザ上で 単体テストを実行するためのテストランナーkarmaをインストール
    "karma": "^1.4.1",
    // karma start時に自動でブラウザが立ち上げ
    "karma-chrome-launcher": "^2.0.0",
    // karmaでmochaを使用するためのプラグイン
    "karma-mocha": "^1.3.0",
    // テスト結果の表示を装飾
    "karma-mocha-reporter": "^2.2.2",
    // JavaScriptのテスティングフレームワークmochaをインストール
    "mocha": "^3.2.0",
    // アサーションライブラリpower-assertをインストール
    "power-assert": "^1.4.2",
    // TypeScript用のwebpackのloaderをインストール
    "ts-loader": "^2.0.0",
    // TypeScriptをインストール
    "typescript": "^2.1.6",
    // モジュールハンドラwebpackをインストール
    "webpack": "^2.2.1",
    // power-assertをwebpack経由で使用するためのloaderをインストール
    "webpack-espower-loader": "^1.0.1"
  },
  "dependencies": {},
  "directories": {
    "test": "test"
  },
  "repository": {
    "type": "git",
    "url": "githubとか"
  },
  "bugs": {
    "url": "githubとか"
  },
  "homepage": "githubのREADMEとか"
}
```

## webpack.config.js
webpack用の設定ファイル

``` 
 module.exports = {
  // ビルドのエントリーポイント（起点）
  entry: {
    // 通常のビルド用
    "indexEntry": './src/ts/index.ts',
    // テスト用のファイルを作成するためのビルド用
    "testEntry": './test/testEntry.ts',
  },
  // 出力先
  output: {
    // 出力先のパス
    path: './dist',
    // 出力するファイルの名前
    // nameには、entryでkeyに指定したエントリーポイントの名前が入る
    filename: '[name].bundle.js'
  },
  // ビルドに必要なモジュールを指定
  module: {
    // loaderを指定
    loaders: [
      // test: ビルド対象を指定, 
      // include: ビルド対象に含める exclude: ビルド対象に含めない
      // loader: 使用するローダー  
      { test: /\.tsx?$/, exclude: /Spec\.ts$/, loader: 'ts-loader' },
      // useは右から使われる
      { test: /Spec\.ts$/, use: ['webpack-espower-loader', 'ts-loader'] },
    ]
  },
  // モジュールの解決に使用するオプション
  resolve: {
    // ビルド対象のファイルの拡張子を配列で指定
    extensions: [".tsx", ".ts", ".js"]
  },
  // devtool : デバッグを楽にするための開発者ツールの登録
  // ビルドしたjavascriptにsource-mapを生成
  devtool: 'inline-source-map',
};
``` 

## karma.conf.js
karmaの設定ファイル

```  
// Karma configuration
module.exports = function (config) {
  config.set({
    // ベースパス
    basePath: '',
    // 使用するテスティングフレームワーク
    frameworks: ['mocha'],
    // karmaでブラウザで読み込むファイル(ここでts->js->bundle後のテストのファイルを指定する)
    files: [
      './dist/testEntry.bundle.js',
    ],
    // list of files to exclude
    exclude: [
    ],
    // テスト結果を装飾するreporterの種類
    reporters: ['mocha'],

    // 使用するポート番号
    port: 9876,

    // 表示する結果をカラーで表示
    colors: true,


    // logのレベル
    // possible values: config.LOG_DISABLE || config.LOG_ERROR || config.LOG_WARN || config.LOG_INFO || config.LOG_DEBUG
    logLevel: config.LOG_DEBUG,


    // テストファイルの変更を自動で監視
    autoWatch: true,

    // テストを実行するブラウザを設定
    browsers: ['Chrome'],

    // Continuous Integration mode
    // if true, Karma captures browsers, runs the tests and exits
    // CIモードにするかどうか、Circle CIでテストを通したかったら、ここをtrueに
    singleRun: true,

    // 同時にいくつのブラウザで実行可能にするかどうかの設定
    concurrency: Infinity
  })
}
``` 

参考にさせていただいたサイト
webpack
http://www.yoheim.net/blog.php?q=20161201
https://webpack.js.org/concepts/
http://qiita.com/tatsuyankmura/items/539c56837fc3a5f258b5
http://dackdive.hateblo.jp/entry/2016/04/13/123000#output
http://thujikun.github.io/blog/2014/12/07/webpack/

karma
http://qiita.com/howdy39/items/b9d704e7f84053924da3
https://karma-runner.github.io/1.0/index.html
http://qiita.com/ken200/items/a2362f91900476e35d07

Mocha 
https://mochajs.org/
http://qiita.com/y_hokkey/items/f73ea6b3d5f6902396b6

power assert
http://azu.github.io/slide/sakurajs/power-assert.html
http://qiita.com/ConquestArrow/items/71e0313ce3eafe411a7d
http://kujira16.hateblo.jp/entry/2016/07/29/215735
