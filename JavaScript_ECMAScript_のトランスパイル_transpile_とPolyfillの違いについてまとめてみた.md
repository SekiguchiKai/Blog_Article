# トランスパイルとPolyfill
各ブラウザのバージョンによって、対応しているJavaScript(ECMAScript)のバージョンは異なる。
そのバージョン対応の差異に対応するためにトランスパイル(transpile)とPolyfilという方法が存在する。
両者の違いがいまいちよくわからなかったので、調べてまとめてみた。

## トランスパイル(transpile)
トランスパイルは一言で言えば、ある言語で書かれたコードを元に別の言語のコードを生成すること。

以下、Wikipediaから引用し、意訳させてもらった（間違っていたらご指摘ください）。
>A source-to-source compiler, transcompiler or transpiler is a type of compiler that takes the source code of a program written in one programming language as its input and produces the equivalent source code in another programming language.
引用元:https://en.wikipedia.org/wiki/Source-to-source_compiler

------
意訳
source-to-source compiler、transcompiler や transpilerは、あるプログラミング言語をインプットとして受けて、同等のソースコードを別のプログラミング言語で出力する。

------
これは、source-to-source compiler、transcompiler や transpilerの説明だが、この「あるプログラミング言語をインプットとして受けて、同等のソースコードを別のプログラミング言語で出力する。」ことがトランスパイル(transpile)。

ECMAScriptでいうと、あるバージョンで記述されたJavaScriptやTypeScriptのファイルを元に別のバージョンのJavaScriptファイルを生成すること。
例えば、ES6で書かれたコードを元にES5のシンタックスにしたコードを生成すること。
TypeScriptのコンパイラやBabelがトランスパイルを行うトランスパイルラー(transpiler)。

こちらは、文法の話。


## Polyfill
Polyfillは一言で言えば、古いブラウザでは実現できないAPIを、古いブラウザでも同等の機能を使用できるように補うための小さなライブラリのようなもの。

最新のAPIを使用してコードを記述しても、Polyfillを使用すれば、そのAPIが対応していない古いバージョンのブラウザでも、その最新のAPIと同等の機能を使用することができるようになる。

以下、Wikipediaから引用し、意訳させてもらった（間違っていたらご指摘ください）。

> In web development, a polyfill is code that implements a feature on web browsers that do not support the feature. Most often, it refers to a JavaScript library that implements an HTML5 web standard, either an established standard (supported by some browsers) on older browsers, or a proposed standard (not supported by any browsers) on existing browsers. Formally, "a polyfill is a shim for a browser API".[1]

>Polyfills allow web developers to use an API regardless of whether it is supported by a browser or not, and usually with minimal overhead. Typically they first check if a browser supports an API, and use it if available, otherwise using their own implementation.[1][2] Polyfills themselves use other, more supported features, and thus different polyfills may be needed for different browsers. The term is also used as a verb: polyfilling is providing a polyfill for a feature.

引用元 https://en.wikipedia.org/wiki/Polyfill

-------
（意訳）
web開発では、Polyfillはある機能をサポートしていないブラウザ上でその機能を実装するためのコードである。
たいていの場合、それはHTML5のwebの標準を実装したJavaScriptのライブラリについて言及していて、その標準とは、古いブラウザ上ですでに確立された標準（いくつかのブラウザ上でサポートされているもの）か、既存のブラウザでは提案段階の標準（どのブラウザにもサポートされていないもの）である。堅い言葉で言えば、「polyfillとは、ブラウザのAPIのshim(詰め木-)-APIの呼び出しを捉えて、その操作などを変更する小さいライブラリ-だ」
(sihimの部分の略の参考サイト：https://en.wikipedia.org/wiki/Shim_(computing) )」


Polyfillによって、webの開発者は、あるAPIをブラウザがサポートしていようがいまいが、大抵の場合は、小さいオーバーヘッドでそれを使えるようになる。一般的にPolyfillは、まずブラウザがそのAPIをサポートしているかを確認し、それが利用できるのならば利用し、さもなければ、Polyfill自身の実装を使用する。Polyfill自身が他のもっとサポートされた機能を使うこともあり、それゆえに他のブラウザのために別のPolyfillが必要となることもあり得る。この（Polyfillという）専門用語は、以下のような動詞としても使用される：
「polyfillingは、ある機能のためにPolypillを提供することだ」

-------

例えば、あるブラウザがPromise APIに対応していない場合に、Polyfillを使用すれば、そのブラウザに対応したPromise APIと互換性を持った違う書き方の実装を提供してくれる。

こちらは、文法ではなく、実行時（ランタイム）の話
こちらはAPIの話

##結局両者の違いは?
トランスパイルは、ある言語のコードを元に別の言語や別のバージョンの言語のファイルを生成すること

polyfillingは、ブラウザでみ対応のAPIに変わる機能を提供する小さなライブラリのこと

## 参考にさせていただいたサイト
https://en.wikipedia.org/wiki/Source-to-source_compiler
https://en.wikipedia.org/wiki/Polyfill
https://en.wikipedia.org/wiki/Shim_(computing))
https://www.d-wood.com/blog/2016/04/12_7917.html
https://azu.github.io/slide-what-is-ecmascript/slide/33.html
http://stackoverflow.com/questions/31205640/what-is-the-difference-between-polyfill-and-transpiler

※ ブログでも同一記事を投稿している
http://www.sekky0905.com/entry/2017/03/05/JavaScript%28ECMAScript%29%E3%81%AE%E3%83%88%E3%83%A9%E3%83%B3%E3%82%B9%E3%83%91%E3%82%A4%E3%83%AB%28transpile%29%E3%81%A8Polyfill%E3%81%AE%E9%81%95%E3%81%84%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6%E3%81%BE
