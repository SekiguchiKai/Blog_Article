## JavaScriptの配列におけるmapとは
配列の全部の要素mapの引数に与えたコールバック関数を適用 => 各要素がここでの結果に変換された配列を生成する。

> map() メソッドは、与えられた関数を配列のすべての要素に対して呼び出し、その結果からなる新しい配列を生成します。

引用元
https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/map

## 実装してみた

``` js
let numArray = [1, 2, 3];
let strArray = numArray.map((move) => {
    switch (move) {
        case 1:
            return 'one';
        case 2:
           return 'two';
        default:
            return  'other';
    }
});
console.log(`map後の配列は、${strArray}`);
```

実行結果

```
map後の配列は、one,two,other
```

## やっていること
mapを通して、numArrayの全要素(コードだとmove)にswitchをしている。

## 実際のコード
わかりにくいので、 `console.log` を `window.alert` に変更している

[Playground · TypeScript](https://www.typescriptlang.org/play/#src=let%20numArray%20%3D%20%5B1%2C%202%2C%203%5D%3B%0D%0Alet%20strArray%20%3D%20numArray.map((move)%20%3D%3E%20%7B%0D%0A%20%20%20%20switch%20(move)%20%7B%0D%0A%20%20%20%20%20%20%20%20case%201%3A%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20return%20'one'%3B%0D%0A%20%20%20%20%20%20%20%20case%202%3A%0D%0A%20%20%20%20%20%20%20%20%20%20%20return%20'two'%3B%0D%0A%20%20%20%20%20%20%20%20default%3A%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20return%20%20'other'%3B%0D%0A%20%20%20%20%7D%0D%0A%7D)%3B%0D%0Aconsole.log(%60map%E5%BE%8C%E3%81%AE%E9%85%8D%E5%88%97%E3%81%AF%E3%80%81%24%7BstrArray%7D%60)%3B#src=let%20numArray%20%3D%20%5B1%2C%202%2C%203%5D%3B%0D%0Alet%20strArray%20%3D%20numArray.map((move)%20%3D%3E%20%7B%0D%0A%20%20%20%20switch%20(move)%20%7B%0D%0A%20%20%20%20%20%20%20%20case%201%3A%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20return%20'one'%3B%0D%0A%20%20%20%20%20%20%20%20case%202%3A%0D%0A%20%20%20%20%20%20%20%20%20%20%20return%20'two'%3B%0D%0A%20%20%20%20%20%20%20%20default%3A%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20return%20%20'other'%3B%0D%0A%20%20%20%20%7D%0D%0A%7D)%3B%0D%0Aconsole.log(%60map%E5%BE%8C%E3%81%AE%E9%85%8D%E5%88%97%E3%81%AF%E3%80%81%24%7BstrArray%7D%60)%3B#src=let%20numArray%20%3D%20%5B1%2C%202%2C%203%5D%3B%0D%0Alet%20strArray%20%3D%20numArray.map((move)%20%3D%3E%20%7B%0D%0A%20%20%20%20switch%20(move)%20%7B%0D%0A%20%20%20%20%20%20%20%20case%201%3A%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20return%20'one'%3B%0D%0A%20%20%20%20%20%20%20%20case%202%3A%0D%0A%20%20%20%20%20%20%20%20%20%20%20return%20'two'%3B%0D%0A%20%20%20%20%20%20%20%20default%3A%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20return%20%20'other'%3B%0D%0A%20%20%20%20%7D%0D%0A%7D)%3B%0D%0Aconsole.log(%60map%E5%BE%8C%E3%81%AE%E9%85%8D%E5%88%97%E3%81%AF%E3%80%81%24%7BstrArray%7D%60)%3B#src=let%20numArray%20%3D%20%5B1%2C%202%2C%203%5D%3B%0D%0Alet%20strArray%20%3D%20numArray.map((move)%20%3D%3E%20%7B%0D%0A%20%20%20%20switch%20(move)%20%7B%0D%0A%20%20%20%20%20%20%20%20case%201%3A%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20return%20'one'%3B%0D%0A%20%20%20%20%20%20%20%20case%202%3A%0D%0A%20%20%20%20%20%20%20%20%20%20%20return%20'two'%3B%0D%0A%20%20%20%20%20%20%20%20default%3A%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20return%20%20'other'%3B%0D%0A%20%20%20%20%7D%0D%0A%7D)%3B%0D%0Aconsole.log(%60map%E5%BE%8C%E3%81%AE%E9%85%8D%E5%88%97%E3%81%AF%E3%80%81%24%7BstrArray%7D%60)%3B#src=let%20numArray%20%3D%20%5B1%2C%202%2C%203%5D%3B%0D%0Alet%20strArray%20%3D%20numArray.map((move)%20%3D%3E%20%7B%0D%0A%20%20%20%20switch%20(move)%20%7B%0D%0A%20%20%20%20%20%20%20%20case%201%3A%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20return%20'one'%3B%0D%0A%20%20%20%20%20%20%20%20case%202%3A%0D%0A%20%20%20%20%20%20%20%20%20%20%20return%20'two'%3B%0D%0A%20%20%20%20%20%20%20%20default%3A%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20return%20%20'other'%3B%0D%0A%20%20%20%20%7D%0D%0A%7D)%3B%0D%0Awindow.alert(%60map%E5%BE%8C%E3%81%AE%E9%85%8D%E5%88%97%E3%81%AF%E3%80%81%24%7BstrArray%7D%60)%3B)

## 参考にさせていただいたサイト
[Array.prototype.map() - JavaScript | MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/map)

[JavaScriptでforEach, filter, map, reduceとか - Qiita](http://qiita.com/itagakishintaro/items/29e301f3125760b81302)
