# package.jsonのdependenciesとdevDependenciesについて~npm~

※ 記載に間違っていた部分がありましたので、修正しました
ご指摘いただいた@chitokuさんありがとうございました

## package.jsonのdependenciesとdevDependenciesとは？
> dependencies
> →実行に必要なパッケージの定義
> 
> devDependencies
> →パッケージの開発に必要なパッケージの定義

以下より、引用
http://nakagaw.hateblo.jp/entry/2015/06/20/204222



## package.jsonへの自動格納方法
npmレジストリに登録された様々な外部ライブラリやパッケージをインストールする時には、以下のoptionをつけてインストールすると、依存関係のあるライブラリやパッケージがpackage.jsonのdependenciesプロパティやdevDependenciesプロパティに追加される

### dependenciesに追加したい場合

```bash
$ npm install —save インストールしたいパッケージ
```

~~もしくは~~ 

~~```
$ npm install -S インストールしたいパッケージ
```~~

### devDependenciesに追加したい場合

```bash
$ npm install --save-dev インストールしたいパッケージ
```

もしくは

```bash
$ npm install -D インストールしたいパッケージ
```

## アンインストールする場合

### dependenciesのパッケージを消し去りたい場合

```bash
$ npm uninstall —save アンインストールしたいパッケージ
```

~~もしくは~~

~~```
$ npm uninstall -S アンインストールしたいパッケージ
```~~

### devDependenciesのパッケージを消し去りたい場合

```bash
$ npm uninstall --save-dev アンインストールしたいパッケージ
```

~~もしくは~~

~~```
$ npm uninstall -D アンインストールしたいパッケージ
```~~

## 参考にさせていただいたサイト
http://qiita.com/sinmetal/items/395edf1d195382cfd8bc
https://docs.npmjs.com/misc/scripts

※ ブログでも同一記事を投稿している
http://www.sekky0905.com/entry/2016/10/22/%E3%80%90package.json%E3%80%91dependencies%E3%81%A8devDependencies%E3%80%90npm%E3%80%91
