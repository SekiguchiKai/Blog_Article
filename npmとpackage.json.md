# npmとpackage.json
#npm
* パッケージ管理ツール
* JavaScript(Node.js)のパッケージやモジュールをインストール、アップデート、共有したりできる
* npmレジストリに登録された様々な外部ライブラリやパッケージをインストールすることができる



#package.json
* 依存関係を記したJSONファイル　　

###機能
* このファイルにプロジェクト毎に必要なパッケージの名前とバージョンを記述すれば、npmが勝手に必要な（依存しているという）パッケージをインストールしてくれる。
* インストールしたパッケージが依存しているパッケージや、さらにそれが依存しているパッケージも自動でインストールしてくれる

###どう使うのか
* `npm install`とpackage.jsonが存在するディレクトリでコマンドを打てば、package.jsonに記述されている依存パッケージを自動でインストールしてくれる
* ```npm install```をするとnode_modulesというディレクトリが```npm install```を実行したディレクトリに作成され、npmを利用してインストールしたパッケージは、この中に格納される

###活用の例
* 別の環境でそのプロジェクトの作業をしたい場合や、他の人がそのプロジェクトの作業をする場合は、そのプロジェクトのGitHub上のpackage.jsonに必要なファイルを記述しておけば、そのプロジェクトを```git clone```し、```npm install```すれば、npmで管理するパッケージに関してはそのまま使えるようになる。

#参考にさせていただいたサイト
http://sekky0905.hatenablog.com/
http://phiary.me/node-js-package-manager-npm-usage/
http://qiita.com/megane42/items/2ab6ffd866c3f2fda066
http://sekky0905.hatenablog.com/
https://www.npmjs.com/

参考文献
わかめ まさひろ（2014）『TypeScriptリファレンス Ver.1.0対応』インプレスジャパン

