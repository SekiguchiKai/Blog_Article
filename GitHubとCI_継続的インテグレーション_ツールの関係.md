# GitHubとCI(継続的インテグレーション)ツールの関係性

「ここ違うよ」とか、「こうした方がいいよ」というのがあったら、ぜひご教授いただけるとありがたいです。

## CI(継続的インテグレーション)及びCI(継続的インテグレーション)ツールとは
そもそもCI(継続的インテグレーション)ツールって何?って方は以下の記事がわかりやすいので参照をオススメする。
[CI(継続的インテグレーション)が解決する課題 | CIツール「Jenkins」活用ソリューション | テクマトリックス株式会社](https://www.techmatrix.co.jp/product/cisolution/cisolution1.html)

[ソフトウェア開発に必須。CI（継続的インテグレーション）のススメ](https://persol-tech-s.co.jp/hatalabo/it_engineer/309.html)

## GitHub
### GitHubのブランチ
GitHubのブランチ運用はGit-flowを前提に以降の記事を記載する。
Git-flowについては、以下の記事がわかりやすい。
[Git-flowって何？ - Qiita](https://qiita.com/KosukeSone/items/514dd24828b485c69a05)

基本的にdevelopブランチから機能やバグフィックス毎にfeatureブランチやhotfixesブランチを生やしていく。

#### developブランチ
> 開発ブランチ。コードが安定し、リリース準備ができたら master へマージする。リリース前はこのブランチが最新バージョンとなる。

引用元 : [Git-flowって何？ - Qiita](https://qiita.com/KosukeSone/items/514dd24828b485c69a05)

また、developブランチには常に正しいコードのみが存在することとする。
(この運用方法については後述する)


#### featureブランチ
(基本的には)機能毎にdevelopブランチから生やして、ここで実際の作業を行う。
[Git-flowって何？ - Qiita](https://qiita.com/KosukeSone/items/514dd24828b485c69a05)を参考にさせていただいた。

#### hotfixesブランチ
バグフィックス作業毎にdevelopブランチから生やして、ここで実際の作業を行う。
[Git-flowって何？ - Qiita](https://qiita.com/KosukeSone/items/514dd24828b485c69a05)を参考にさせていただいた。

## GitHubとCI(継続的インテグレーション)ツール
### GitHubとCIツールの関係性と運用方法
* developブランチは、常に最新で正しいコードが置かれていることとする
* featureブランチで機能実装やhotfixesブランチでバグフィックスの作業を行う
* featureブランチやhotfixesブランチで作業した結果のコードをCIツールでコードをチェックする
* 機能実装やバグフィックスが完了した上で、CIツールでのチェックがこけなかったら、レビュー者にpull/requestを送信する
* レビュー者がレビューし、問題がなければdevelopブランチにfeatureブランチやhotfixesブランチをmergeする

以下、図にしてみた
![GitHubとCIツール.png](https://qiita-image-store.s3.amazonaws.com/0/145611/cea1b213-14b1-1feb-1190-3cd05837d01f.png)


### Gitでの開発にCIツールを導入するメリット
* リリースされるプロダクトの品質を管理することができる。
=> developブランチに常に正しいコードのみが存在する状態にでき、git-flowではdevelopからリリース用のmasterブランチにmergeするため、常にCIツール的に正しいコードのみがリリースされることとなる
* 機能実装やバグフィックスを行なった時に、デグレーションが起きないことを保証する
* テストや静的解析解析ツールを自動化することができ、属人化の排除や効率化を行うことができる

### 流れ
#### featureブランチを生やすところから開発の流れ
1. developブランチからfeatureブランチを作成する
2. 作成したブランチ内で、作業を行う(実装など)
3. こまめに `git add` 、 `git commit ` `git push origin hoge` を行う

#### mergeの流れ
1. pushすると、CIツールが自動でテストを行なったり、静的解析解析をしたりして、pushされたコードが正しいコードかどうかを確認する。<br>
=> ここで、正しくない場合は、CIツールがこける(fail)

#### featureブランチの機能が完成した上で、pushしたコードが正しかった場合(CIツールがこけなかった場合)
1. レビュー者に対して、pull/requestを送る
2. レビュー者がコードをレビューし、問題がなく承認すればレビュー者がpull/requestを送信したfeatureブランチをdevelopブランチぶmergeする

#### featureブランチの機能が完成した上で、pushしたコードが正しくなかった場合(CIツールがこけた場合)
1. developにはmergeしない
=> developには常に正しいコードのみがmergeされるべきだからである
2. 再度、CIツールでこけた部分の修正を行い、pushを行う

## まとめ
GitHubとCIツールを連携させて、継続的インテクレーションを行えば、テストや静的解析を自動化することができ、効率的に持続可能なデグレ防止や品質管理を自動で行うことができるので、最初は手間でもぜひ導入すべき!

## 関連記事
[Circle CI 2.0の基礎的な設定まとめてみた(GAE/Goのサンプル付き) - Qiita](https://qiita.com/Sekky0905/items/7f9aa94261e17e4fd040)

## 参考にさせていただいたサイト
### GitHub
[Git超絶まとめ - Qiita](https://qiita.com/masashi127/items/2e103c3fba9d1b058961)
[Git-flowって何？ - Qiita](https://qiita.com/KosukeSone/items/514dd24828b485c69a05)

### CIツール
[CI(継続的インテグレーション)が解決する課題 | CIツール「Jenkins」活用ソリューション | テクマトリックス株式会社](https://www.techmatrix.co.jp/product/cisolution/cisolution1.html)
[ソフトウェア開発に必須。CI（継続的インテグレーション）のススメ](https://persol-tech-s.co.jp/hatalabo/it_engineer/309.html)



