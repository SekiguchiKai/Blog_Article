# Goでコンストラクタ

オブジェクト指向の言語でよく用いられるコンストラクタ<br>
Goでもそのコンストラクタを書くことが可能だということで書いてみた

``` go
import "fmt"

type Language struct {
     Name     string
     LangType string
}

// コンストラクタ
// 戻り値として返すのは、構造体のポイントであることに注意
func NewLanguage(name string, langType string) *Language {
     // コンストラクタの関数内で、構造体をnew
     l := new(Language)
     // 以下、構造体の各フィールドを引数で受け取った値に設定
     l.Name = name
     l.LangType = langType
     // 構造体のインスタンスを返す
     return l
}

func main() {
     l := NewLanguage("Go", "Static")
     fmt.Println("名前" + l.Name)
     fmt.Println("言語" + l.LangType)
}
```

## 注意点
1. コンストラクタ内で、対象の構造体をnewする
2. フィールドを設定した構造体を戻り値として返す 


## 他の言語では...
ちなみにTypeScriptのコンストラクタはこんな感じなので、だいぶ違う<br>

``` js
class Language { 
    constructor(
    private _name: string,
    private _langType: string) {}

    get name(): string {
        return this._name;
    }

    get langType(): string{ 
        return this._langType;
    }
}

var lang = new Language('TypeScript', 'Static');
console.log(`名前:${lang.name}`);
console.log(`名前:${lang.langType}`); 

```

## 参考文献
* 松尾 愛賀　(2016/4/15)『スターティングGo言語』 翔泳社

※ ブログでも同一記事を書いている
http://www.sekky0905.com/entry/2017/02/23/Go%E3%81%A7%E3%82%B3%E3%83%B3%E3%82%B9%E3%83%88%E3%83%A9%E3%82%AF%E3%82%BF
