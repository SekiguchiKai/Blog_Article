# TypeScriptで document.getElementById("hoge").value をすると出るThe property ‘hoge' does not exist on value of type 'HTMLElement' というエラーを解消する

TypeScriptでElementの値を ``` document.getElementById("hoge").value ``` で取得しようとしたところエラーが出た。


##症状
``` document.getElementById(‘hoge’).value ``` を使用すると、以下のエラーが生じる。

発生したエラー
``` The property ‘hoge' does not exist on value of type 'HTMLElement' ```

---
(意訳)‘hoge'というプロパティは ```HTMLElement ```型の値には存在しない。

---
##エラーになる理由

``` document.getElementById(‘hoge’); ```の戻り値は、HTMLElement型で、エラーの文章にもある通り、この型のインターフェースにはvalueプロパティは存在しない

HTMLElementインターフェースのドキュメント
https://developer.mozilla.org/ja/docs/Web/API/HTMLElement

##どうすれば良いか

HTMLInputElementというHTMLElementを継承したインターフェースには、valueプロパティが存在するので、こちらにType Assertionすれば良い

HTMLElementインターフェースのドキュメント
https://developer.mozilla.org/ja/docs/Web/API/HTMLInputElement

```ts
const element: HTMLInputElement =<HTMLInputElement>document.getElementById(‘hoge');
const value: string = element.value;
```

HTMLInputElementインターフェースのドキュメント
https://developer.mozilla.org/ja/docs/Web/API/HTMLInputElement

## 参考にさせていただいたサイト
[javascript - The property 'value' does not exist on value of type 'HTMLElement' - Stack Overflow](https://stackoverflow.com/questions/12989741/the-property-value-does-not-exist-on-value-of-type-htmlelement)
https://developer.mozilla.org/ja/docs/Web/API/HTMLElement
https://developer.mozilla.org/ja/docs/Web/API/HTMLInputElement


