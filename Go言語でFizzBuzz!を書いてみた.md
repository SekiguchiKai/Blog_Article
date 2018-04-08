# Go言語でFizzBuzz!
Go言語でFizzBuzz!を書いてみた

## ルール
1~100までの数字で、
3で割り切れれば、「Fizz!」を表示する
5で割り切れれば、「Buzz!」を表示する
3と5で割り切れれば、「Fizz Buzz!」を表示する
上記以外の場合は、そのままの数字を表示する

## いざコード
``` go
package main

import (
	"fmt"
)

func main() {

	i := 1
	for i < 101 {
		switch {
		case i%15 == 0:
			fmt.Println("FIZZ BUZZ!")
		case i%3 == 0:
			fmt.Println("FIZZ!")
		case i%5 == 0:
			fmt.Println("BUZZ!")
		default:
			fmt.Println(i)
		}

		i++
	}

}
```

## 結果

```
1
2
FIZZ!
4
BUZZ!
FIZZ!
7
8
FIZZ!
BUZZ!
11
FIZZ!
13
14
FIZZ BUZZ!
16
17
FIZZ!
19
BUZZ!
FIZZ!
22
23
FIZZ!
BUZZ!
26
FIZZ!
28
29
FIZZ BUZZ!
31
32
FIZZ!
34
BUZZ!
FIZZ!
37
38
FIZZ!
BUZZ!
41
FIZZ!
43
44
FIZZ BUZZ!
46
47
FIZZ!
49
BUZZ!
FIZZ!
52
53
FIZZ!
BUZZ!
56
FIZZ!
58
59
FIZZ BUZZ!
61
62
FIZZ!
64
BUZZ!
FIZZ!
67
68
FIZZ!
BUZZ!
71
FIZZ!
73
74
FIZZ BUZZ!
76
77
FIZZ!
79
BUZZ!
FIZZ!
82
83
FIZZ!
BUZZ!
86
FIZZ!
88
89
FIZZ BUZZ!
91
92
FIZZ!
94
BUZZ!
FIZZ!
97
98
FIZZ!
BUZZ!
```

※ ブログでも同一記事を投稿している
http://www.sekky0905.com/entry/2017/02/26/Go%E8%A8%80%E8%AA%9E%E3%81%A7FizzBuzz%21%E3%82%92%E6%9B%B8%E3%81%84%E3%81%A6%E3%81%BF%E3%81%9F
