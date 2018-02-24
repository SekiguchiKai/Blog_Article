# webpack&モジュールバンドラ(Module Bundler)&ローダー(Loader)概要

# webpack&モジュールバンドラ&ローダー

最近、webpackを使用した<br>
その時に、webpackとモジュールバンドラ、ローダーの基本的なことについて学んだので、まとめようと思う<br>

なお、本記事では、基本的な概念や使用する際の注意事項のメモが中心となるので、設定ファイルの説明などは行っていない<br>

## モジュールバンドラ(Module Bundler)
### モジュールバンドラ(Module Bundler)とは
* 複数に分割されたJavaScriptのファイル（Module）を結びつけて1つのJavaScriptファイルにするツール

### なぜモジュールバンドラ(Module Bundler)が必要か
* 現在、最新のECMAScriptでは、1ファイル=1モジュールの考え方が考案され徐々に広まってきている
* しかし、現存のほとんどのブラウザでは、それに対応しておらず、scriptタグに書かれたJavaScriptのみを実行するのが普通だ
* モジュールバンドラ(Module Bundler)を使用すれば、複数のJavaScriptファイルをまとめ上げて1つのJavaScriptファイルに統合してくれる
* また、HTMLにエントリポイントとなるJavaScriptファイルのみを貼っておいても、現在のブラウザではモジュール間の依存関係を解決して、他のモジュールまで利用するということはできない（複数のJavaScriptファイル間の依存関係を解決してくれない）
* そこで、モジュールバンドラ(Module Bundler)は、複数に分割されたJavaScripファイル(モジュール)を1つのJavaScriptファイルにしてしまい、さらにその際に複数のJavaScriptファイル(Module)間の依存関係を解決してくれる


## webpack

### webpackとは
* モジュールバンドラ

### webpackの機能（ざっくり）
1. 分割したJavaScriptファイルを1つにまとめ上げられる（1つのファイルにビルドできる）=>bundle.jsになる
2. JavaScript同士の依存関係も解決してくれる

参考にさせていただいたサイト<br>
https://webpack.js.org/
https://ics.media/entry/12140
http://qiita.com/ossan-engineer/items/8352bdeab9ce8c8c00ef
http://qiita.com/chuck0523/items/caacbf4137642cb175ec

さらにwebpackは複数のbundle.jsをアウトプットすることもできる<br>
=> 例えばテストには、テスト用のtest_bundle.jsを作ることができる<br>
=>テストにおいても、JavaScriptはモジュールの分割の依存関係を解消するために、バンドルする必要があるのでそれに役立つ<br>

複数の出力ファイルをバラバラに出力するのに参考にさせていただいたサイト<br>
http://webdesign-dackel.com/2015/09/10/webpack-multiple-output/

### webpackで注意すべきこと
* webpackは循環依存は解決できない<br>
=> 例えば、エントリポイントであるA.jsとB.jsがお互いに依存しあっている場合、ループしてしまう<br>
=>index.js(エントリポイントとなるファイル)はimportできない<br>

                              
## loader
* loaderとは、JavaScriptのファイル中でimportした静的リソースをブラウザで扱える形に変換してくれるものである
* もっと詳しく言えば、loaderに対してパラメータ(引数)としてリソースファイルを与えると、JavaScriptで扱えるファイルに変換して返してくれる<br>
=> 例えば、ts-loaderを使用すると、TypeScriptのファイルでimportしたファイルをjsファイルに変換してくれる<br>

参考にさせていただいたサイト<br>
http://qiita.com/chuck0523/items/caacbf4137642cb175ec
http://qiita.com/howdy39/items/48d85c430f90a21075cd
https://webpack.js.org/concepts/loaders/(公式)


## おまけ : TypeScript with webpack

### TypeScriptでのwebpackの流れ
1. TypeScriptに対して、ts-loaderを使い、内部的に個別のjsファイルを作成する<br>
例)A.ts, B.ts, C.tsが存在するとき、ts-loaderを使用すると、A.js, B.js, C.jsがwebpack内部で生成される<br>

2. web packの本来の機能を使用して、内部的に生成された複数の.jsファイルを1つのbundle.jsにまとめ上げる<br>


### TypeScriptでwebpackを使用する際の諸注意
* 複数のJavaScriptファイルを生成しなくても、bundle.jsを生成すれば良い
* 例えば、Gi tHubでソースコードを管理している場合、基本的には、後で再生成可能なファイルは、Git Hub上に置かないほうがよしとされている
=> 万が一、複数のjsファイルを生成しても、それは再生成可能なファイルなので、.gitignoreするといいかもしれない


※ 同一内容を個人のブログでも書いている
http://www.sekky0905.com/entry/2017/02/19/%E3%80%90webpack%E3%80%91webpack%26%E3%83%A2%E3%82%B8%E3%83%A5%E3%83%BC%E3%83%AB%E3%83%90%E3%83%B3%E3%83%89%E3%83%A9(Module_Bundler)%26%E3%83%AD%E3%83%BC%E3%83%80%E3%83%BC(Loader)%E6%A6%82%E8%A6%81
