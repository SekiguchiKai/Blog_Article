
# SendGridでメールを送信する時に改行が上手くいかない時の対処法

SendGridを介してメールを送信する時に、Contentをplain/textにしていると意図した改行がなされないとことがある。その時の対処法を記す。
対処法と言っても、非常に簡単な設定を行うだけである。

## 生じる問題

例えば、以下のような改行コードを入れた文字列の場合に生じる(SendGridのAPIを叩くためのJSONの一部を抜粋)

```
Content: []Contents{{
       Type:  "text/plain”,       
       Value: "アイウエオ\nカキクケコ\nサシスセソ",}},
```

この場合、メールで送信されてくる中身は、以下のようなものを期待している

![expected_mail_screen.png](https://qiita-image-store.s3.amazonaws.com/0/145611/9520e59c-d2a8-527e-9512-f6c43e4887ad.png)

しかし、デフォルトの設定のままだと以下のような内容が送信されてくる

![actual_mail_screen.png](https://qiita-image-store.s3.amazonaws.com/0/145611/662b47a8-99c3-4628-86a6-2402dd1bff0e.png)

## 問題が生じる原因
SendGridの `Plain Content: Convert your plain text emails to HTML` .という設定が、OFFになっているために生じる。(Settings > Mail Settings > Plain ContentのACTIVE)
これは、その設定の説明の通り、plain text のメールをSendGridがHTMLに変換してくれるというものである。
これがあるために、改行コードが無視されたHTML形式に変換されメールが送信されるので、先ほどのような問題が生じる。

![plain_text_off.png](https://qiita-image-store.s3.amazonaws.com/0/145611/2cc3eafc-bbb4-d8dc-7065-77126a27a001.png)


## 解決策
`Plain Content: Convert your plain text emails to HTML` (Settings > Mail Settings > Plain ContentのACTIVE)の設定を「ON」にすれば良い
若干ややこしいが、「OFF」にしていると、plain text のメールをSendGridがHTMLに自動で変換されるようになってしまう。

![plain_text_on.png](https://qiita-image-store.s3.amazonaws.com/0/145611/a60ce48e-abc7-6dec-ce8c-09d0238cf86e.png)


## 関連記事
 [Go言語でSendGrid Web API v3を使って、メールを送信する](http://qiita.com/Sekky0905/items/7f7285d0a2b3e5f21e57 "Go言語でSendGrid Web API v3を使って、メールを送信する")


 [GCE上のGoのコードからSendGridを使用してメールを送信](http://qiita.com/Sekky0905/items/ce9588b29ac9fca507c2 "GCE上のGoのコードからSendGridを使用してメールを送信")
