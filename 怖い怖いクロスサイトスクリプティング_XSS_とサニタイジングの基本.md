# クロスサイトスクリプティング(XSS)とサニタイジングの基本

## クロスサイトスクリプティング(XSS)とは
* 動的なwebを作成する時に、入力画面が発生する。
* その入力画面（ form要素）に、JavaScriptやHTMLの文字列を埋め込んで、悪意のある動作をさせることである。

##　対策
* このようなクロスサイトスクリプティングを対策するためには、以下のHTML特殊文字をエスケープさせることが必要になる

``` 
「<」「>」「&」「"」「’」
```
* 上記の特殊文字はその名の通り、HTMLにとって特別な意味を持つ。
* なので、上記の特殊文字を特別な意味のない普通の文字列に変換する必要がある。
* このような、特殊文字を一般的な文字列に変換する処理をサニタイジングという。
（参考：http://www.weblio.jp/content/%E3%82%B5%E3%83%8B%E3%82%BF%E3%82%A4%E3%82%BA）

### 具体的な置き換え
先ほどのHTMLにとって特別な意味を持つ記号を意味のない文字に置き換える<br>
以下が具体例<br>

```
& => "&amp;" 
< => "&lt;" 
> => "&gt;"
" => “&quot;”
' => &#x27; 
\=> "&#x2F;"
```
(引用元)
https://www.owasp.org/index.php/XSS_%28Cross_Site_Scripting%29_Prevention_Cheat_Sheet

### サニタイジングを行うタイミング
* HTMLを出力する直前にエスケープを行う
=>文字列を受け取った直後にエスケープすると、その文字列を使用して行う処理に異常が発生する可能性があるから


## 参考にさせていただいたサイト
http://viral-community.com/blog/xss-1835/
http://www.atmarkit.co.jp/ait/articles/0603/23/news119.html
https://www.owasp.org/index.php/XSS_%28Cross_Site_Scripting%29_Prevention_Cheat_Sheet
http://codezine.jp/article/detail/9149?p=3
http://javatechnology.net/java/html-sanitizing/

※ ブログでも同一記事を投稿している
http://www.sekky0905.com/entry/2017/02/25/%E6%80%96%E3%81%84%E6%80%96%E3%81%84%E3%82%AF%E3%83%AD%E3%82%B9%E3%82%B5%E3%82%A4%E3%83%88%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%97%E3%83%86%E3%82%A3%E3%83%B3%E3%82%B0%28XSS%29%E3%81%A8%E3%82%B5%E3%83%8B%E3%82%BF
