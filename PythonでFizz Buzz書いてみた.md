# PythonでFizz Buzz書いてみた
静的言語ばかりやってきたが、機械学習とかやりたいので、Pythonの勉強を始めることにした。
これまでやってきたGo、Java、TypeScriptとはなかなか違うところが多くてカルチャーショックを受けている...
とは言え、新しい言語を学ぶのは心が躍るものだ。
とりあえず、基礎の基礎ということで、Fizz Buzzを書いてみた。

## ルール
1~100までの数字で、
3で割り切れれば、「Fizz!」を表示する
5で割り切れれば、「Buzz!」を表示する
3と5で割り切れれば、「Fizz Buzz!」を表示する
上記以外の場合は、そのままの数字を表示する

## いざコード

### Whileバージョン

```py3
i = 1
while i < 101:
    if i % 15 == 0:
        print("Fizz Buzz!")
    elif i % 3 == 0:
        print("Fizz!")
    elif i % 5 == 0:
        print("Buzz!")
    else:
        print(i)

    i += 1
```

### forバージョン

```py3
for i in range(1, 101):
    if i % 15 == 0:
        print("Fizz Buzz!")
    elif i % 3 == 0:
        print("Fizz!")
    elif i % 5 == 0:
        print("Buzz!")
    else:
        print(i)
```

## 実行結果

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

## 関連記事
[Go言語でFizzBuzz!を書いてみた - Qiita](http://qiita.com/Sekky0905/items/9a5a459e143303cd0a18)
[TypeScript(JavaScript)でアロー関数で即時関数でFizzBuzz! - Qiita](http://qiita.com/Sekky0905/items/e5523901205e7ccd20ef)

※ ブログでも同一の投稿を行っている
[PythonでFizz Buzz書いてみた - sekky0905’s blog](http://www.sekky0905.com/entry/2017/07/23/Python%E3%81%A7Fizz_Buzz%E6%9B%B8%E3%81%84%E3%81%A6%E3%81%BF%E3%81%9F)
