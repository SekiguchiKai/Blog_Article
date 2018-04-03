# TypeScriptでアロー関数で即時関数でFizzBuzz!
TypeScriptでアロー関数と即時関数を使ってFizzBuzz!を書いてみた

## ルール
1~100までの数字で、
3で割り切れれば、「Fizz!」を表示する
5で割り切れれば、「Buzz!」を表示する
3と5で割り切れれば、「Fizz Buzz!」を表示する
上記以外の場合は、そのままの数字を表示する

## いざコード

``` js
(() => {
    for (let i = 1; i < 101; i++) {
        if (i % 15 === 0) {
            console.log('Fizz Buzz!');
        } else if (i % 3 === 0) {
            console.log('Fizz!');
        } else if (i % 5 === 0) {
            console.log('Buzz!');
        } else {
            console.log(i);
        }
    }
})();
```


## 結果

```
1
2
Fizz!
4
Buzz!
Fizz!
7
8
Fizz!
Buzz!
11
Fizz!
13
14
Fizz Buzz!
16
17
Fizz!
19
Buzz!
Fizz!
22
23
Fizz!
Buzz!
26
Fizz!
28
29
Fizz Buzz!
31
32
Fizz!
34
Buzz!
Fizz!
37
38
Fizz!
Buzz!
41
Fizz!
43
44
Fizz Buzz!
46
47
Fizz!
49
Buzz!
Fizz!
52
53
Fizz!
Buzz!
56
Fizz!
58
59
Fizz Buzz!
61
62
Fizz!
64
Buzz!
Fizz!
67
68
Fizz!
Buzz!
71
Fizz!
73
74
Fizz Buzz!
76
77
Fizz!
79
Buzz!
Fizz!
82
83
Fizz!
Buzz!
86
Fizz!
88
89
Fizz Buzz!
91
92
Fizz!
94
Buzz!
Fizz!
97
98
Fizz!
Buzz!
```

※ ブログでも同一記事を投稿している
http://www.sekky0905.com/entry/2017/02/26/TypeScript%E3%81%A7%E3%82%A2%E3%83%AD%E3%83%BC%E9%96%A2%E6%95%B0%E3%81%A7%E5%8D%B3%E6%99%82%E9%96%A2%E6%95%B0%E3%81%A7FizzBuzz%21
