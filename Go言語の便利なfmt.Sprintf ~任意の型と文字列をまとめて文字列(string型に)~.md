# Go言語の便利な`fmt.Sprintf` ~任意の型と文字列をまとめて文字列(string型に)~
Goの静的型付けにおいて、`string`と他の型を`string`型として一緒に扱うことができるようにする`fmt.Sprintf`という便利な関数が存在する。

## Goの書式指定子

Go言語は、静的型付けの言語あり、`string`と他の型は一緒には扱えない。
そのため、[書式指定子](http://qiita.com/Sekky0905/items/c9cbda2498a685517ad0 "go 書式指定子") が存在する。

例えば以下のようにできる。

``` go
package main

import (
	"fmt"
)

func main() {
	type Person struct {
		Age    int
		Height int
	}

	p := new(Person)
	p.Age = 26
	p.Height = 170

	fmt.Printf("年齢%d\n身長%d", p.Age, p.Height)
}

```

結果

```
年齢26
身長170
```
## `fmt.Sprintf` を使う

上記のコードは、`string`と構造体`Person`のフィールドを一緒に扱っていた。

書式指定子で、任意の型と`string`を一緒に扱えるようになったはいいが、それ自体を文字列(`string`)として表すことはできないだろうか。
そんな時に活躍するのが、 [`fmt.Sprintf`](https://golang.org/pkg/fmt/#Sprintf "`fmt.Sprintf`")である。

`fmt.Sprintf` は、書式指定子に沿ってフォーマットした文字列を返してくれる。


``` go
package main

import (
	"fmt"
)

func main() {
	type Person struct {
		Age    int
		Height int
	}

	p := new(Person)
	p.Age = 26
	p.Height = 170

	str := fmt.Sprintf("年齢:%d\n身長:%d", p.Age, p.Height)
	fmt.Println(str)

}

```

結果

```
年齢:26
身長:170
```

上記のコードでは、 `"年齢%d\n身長%d"` という文字列(string)と `p.Age`, `p.Height` という`int`型が書式指定子によってフォーマットされ、それらが結合された文字列が`string`型として、戻り値で`str`に格納されている。

## もう少し応用してみる

以下のようなことも可能だ。

``` go
package main

import (
	"fmt"
)

func main() {
	type Person struct {
		Age    int
		Height int
	}

	p := new(Person)
	p.Age = 26
	p.Height = 170

	str := fmt.Sprintf("年齢:%d\n身長:%d", p.Age, p.Height)
	fmt.Println(str + "\n" + "ハンドルネーム:Sekky0905")

}

```

上記のコードは、一旦`fmt.Sprintf`で`string`　と`int`をフォーマットし、`string`にまとめてstrに格納している。

`str`は`string`型なので、`fmt.Println(str + "\n" + "ハンドルネーム:Sekky0905")`というのが可能になっている。

## 参考にさせていただいたサイト
[`fmt.Sprintf`](https://golang.org/pkg/fmt/#Sprintf "`fmt.Sprintf`")

## 関連記事
[忘れがちなGo言語の書式指定子を例付きでまとめた](http://qiita.com/Sekky0905/items/c9cbda2498a685517ad0 "忘れがちなGo言語の書式指定子を例付きでまとめた")



※ ブログでも同一記事を投稿している
[Go言語の便利なfmt.Sprintf ~任意の型と文字列をまとめて文字列(string型に)~](http://www.sekky0905.com/entry/2017/05/19/ "Go言語の便利なfmt.Sprintf ~任意の型と文字列をまとめて文字列(string型に)~")

