# 忘れがちなGo言語の書式指定子を例付きでまとめた

# Goの書式指定子

忘れがちな書式指定子<br>
いちいち調べるのも面倒臭いので、普段使いそうな書式指定子の使い方を例付きでまとめた<br>

各書式指定子の説明文は、基本的に以下のサイトを引用させていただいている
引用元: [go fmtパッケージ](http://golang.jp/pkg/fmt "go fmtパッケージ")


## 各書式指定子の使い方

### %v デフォルトフォーマット
> %v  デフォルトフォーマットを適用した値
      構造体を出力する際、+フラグ(%+v)を加えるとフィールド名が表示される

``` go
package main

import (
    "fmt"
)

type Person struct {
    Name     string
    Favorite string
}

func main() {
    p := new(Person)
    p.Name = "sekky"
    p.Favorite = "Programming"
    fmt.Printf("構造体 = %v", p)
}

```

結果

``` go
構造体 = &{sekky Programming}
``` 

---

### %+v フラグ付き デフォルトフォーマット
> フラグ(%+v)を加えるとフィールド名が表示される

``` go
package main

import (
    "fmt"
)

type Person struct {
    Name     string
    Favorite string
}

func main() {
    p := new(Person)
    p.Name = "sekky"
    p.Favorite = "Programming"
    fmt.Printf("構造体 = %+v", p)
}

```
結果

``` go
構造体 = &{Name:sekky Favorite:Programming}
```

---


### %#v Go言語の構文で表現
> %#v この値をGo言語の構文で表現する

``` go
package main

import (
    "fmt"
)

type Person struct {
    Name     string
    Favorite string
}

func main() {
    p := new(Person)
    p.Name = "sekky"
    p.Favorite = "Programming"
    fmt.Printf("構造体 = %#v\n", p)

    a := [...]string{"Go", "Java", "Python", "Ruby", "Rust"}
    fmt.Printf("配列 = %#v", a)
}

```

結果

``` go
構造体 = &main.Person{Name:"sekky", Favorite:"Programming"}
配列 = [5]string{"Go", "Java", "Python", "Ruby", "Rust"}
``` 

---


### %T 値の型
> %T  この値の型をGo言語の構文で表現する

``` go 
package main

import (
    "fmt"
)

type Person struct {
    Name     string
    Favorite string
}

func main() {
    p := new(Person)
    p.Name = "sekky"
    p.Favorite = "Programming"
    fmt.Printf("構造体 = %T\n", p)

    a := [...]string{"Go", "Java", "Python", "Ruby", "Rust"}
    fmt.Printf("配列 = %T\n", a)

    s := []int{0, 1, 2}
    fmt.Printf("スライス = %T\n", s)

    n := 7
    fmt.Printf("int = %T\n", n)

    str := "sekky"
    fmt.Printf("string = %T\n", str)

    b := true
    fmt.Printf("bool = %T\n", b)
}
```
結果

``` go
構造体 = *main.Person
配列 = [5]string
スライス = []int
int = int
string = string
bool = bool
```

---


### %t 単語、trueまたはfalse
> %t  単語、trueまたはfalse

``` go
package main

import (
    "fmt"
)

type Person struct {
    Name     string
    Favorite string
}

func main() {

    b := true
    fmt.Printf("%t\n", b)

    b = false
    fmt.Printf("%t\n", b)
    
    str := "Go"
    fmt.Printf("%t\n", str)
    
    n := 1
    fmt.Printf("%t\n", n)
    
    p := new(Person)
    p.Name = "sekky"
    fmt.Printf("%t\n", p)
    
    
}
```

結果

``` go
true
false
%!t(string=Go)
%!t(int=1)
&{%!t(string=sekky) %!t(string=)}
``` 

---


### %s 文字列、スライス
> %s  文字列またはスライスそのまま

``` go
package main

import (
    "fmt"
)

func main() {

    str := "Go"
    fmt.Printf("文字列 = %s\n", str)

    s := []string{"Go", "Java"}
    fmt.Printf("スライス = %s", s)

    // 配列では使用できないため、以下のコードはエラーになる
    // s := [2]string{"Go", "Java"}
    // fmt.Printf("配列 = %s", s)

}
```

結果

``` go
文字列 = Go
スライス = [Go Java]
``` 

---


### 数値系
数値系はまとめて書く
コメント部分は以下を引用 
 引用元 : [go fmtパッケージ](http://golang.jp/pkg/fmt "go fmtパッケージ")<br>


``` go
package main

import (
    "fmt"
)

func main() {

    // %b 基数2
    b := 5
    fmt.Printf("基数2 = %b\n", b)

    // %c 対応するUnicodeコードポイントによって表される文字
    c := '\101'
    fmt.Printf("Unicodeコードポイント = %c\n", c)

    // %d 基数10
    d := 5
    fmt.Printf("基数10 = %d\n", d)

    // %o 基数8
    o := 20
    fmt.Printf("基数8 = %o\n", o)

    // %x 基数16、10以上の数には小文字(a-f)を使用
    x := 2059
    fmt.Printf("基数16、10以上の数には小文字(a-f)を使用 = %x\n", x)

    // %X 基数16、10以上の数には大文字(A-F)を使用
    X := 2059
    fmt.Printf("基数16、10以上の数には大文字(A-F)を使用 = %X\n", X)

    // %U ユニコードフォーマット: U+1234; "U+%x"と同じ。デフォルトは、4桁
    U := '\101'
    fmt.Printf("ユニコードフォーマット = %U\n", U)

}
```

結果

``` go
基数2 = 101
Unicodeコードポイント = A
基数10 = 5
基数8 = 24
基数16、10以上の数には小文字(a-f)を使用 = 80b
基数16、10以上の数には大文字(A-F)を使用 = 80B
ユニコードフォーマット = U+0041
```

<hr>


### ポインタ
> %p  16進数文字列、先頭に0x

``` go
package main

import (
    "fmt"
)

type Person struct {
    Name     string
    Favorite string
}

func main() {
    p := new(Person)
    p.Name = "sekky"
    p.Favorite = "Programming"
    fmt.Printf("構造体 = %p", p)
}
```

結果

``` go
構造体 = 0x1040a130
```

## 参考文献
松尾 愛賀　(2016/4/15)『スターティングGo言語』 翔泳社

## 参考にさせていただいたサイト
 [go fmtパッケージ](http://golang.jp/pkg/fmt "go fmtパッケージ")

## ブログでも同一記事を投稿している
 [忘れがちなGo言語の書式指定子を例付きでまとめた - sekky0905’s blog〜文系元営業プログラミング未経験からエンジニアへ〜](http://www.sekky0905.com/entry/2017/05/05/%E5%BF%98%E3%82%8C%E3%81%8C%E3%81%A1%E3%81%AAGo%E8%A8%80%E8%AA%9E%E3%81%AE%E6%9B%B8%E5%BC%8F%E6%8C%87%E5%AE%9A%E5%AD%90%E3%82%92%E4%BE%8B%E4%BB%98%E3%81%8D%E3%81%A7%E3%81%BE%E3%81%A8%E3%82%81%E3%81%9F "忘れがちなGo言語の書式指定子を例付きでまとめた - sekky0905’s blog〜文系元営業プログラミング未経験からエンジニアへ〜")
