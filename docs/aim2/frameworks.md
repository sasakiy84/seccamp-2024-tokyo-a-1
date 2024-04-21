# フレームワークという登場人物

ここでは、いくつかのフレームワークやライブラリにおいて、CSRF の対策がどのように行われているかを確認します。
これを通して、今まで見てきた CSRF のどの対策が現実で使われているのかや、フレームワークを使う理由のひとつを確認してもらえればと思います。

## Ruby on Rails

広く使わている Web アプリを作るためのフレームワークです。チュートリアルが非常に充実しており、勉強になります。

まずドキュメントには、GET と POST を適切に使い分けましょうと書いてあります。

Rails では、コントローラーと呼ばれる、リクエストを受け取る最外部に、以下のコードを追加するだけで、csrf token を自動で追加し、検証してくれるようです。また、デフォルトでこの設定が使われているようです。

```
protect_from_forgery with: :exception
```

https://railsguides.jp/security.html#csrf%E3%81%B8%E3%81%AE%E5%AF%BE%E5%BF%9C%E7%AD%96

具体的には、csrf token をフォームへ埋め込み、かつトークンをセッションに埋め込みます。
そしてフォーム送信時にサーバーで突き合わせる方法を取っているようです。
セッションはデフォルトでは文字列として暗号化され、cookie に保存されるようになっています。

他にも、Origin のチェック機能などを提供しているようです。
https://api.rubyonrails.org/v6.0.2.2/classes/ActionController/RequestForgeryProtection.html#method-i-valid_request_origin-3F

## Laravel

PHP のフレームワークです。
Laravel では、セッションごとに csrf token が生成されます。フォームの場合は、そのトークンを使うことを明示的に宣言してトークンをフォームに埋め込み、 CSRF の対策を行うようです。
宣言の方法は、`@csrf`を書くだけなので、楽ですね。検証はミドルウェアが自動でしてくれるようです。
https://laravel.com/docs/11.x/csrf


## Express

Node.js の有名なフレームワークです。この本のサンプルアプリにも使われています。最小限のフレームワークという謳い文句を掲げています。

最小限のフレームワークという通り、デフォルトでは CSRF の対策は行っていません。ではどうするかというと、ミドルウェアと呼ばれるプラグインのようなものを追加するか、サンプルアプリでやっているように自分で追加することになります。

追加できるミドルウェアのうち、公式がメンテナンスをしているものがありました。しかし、公式のリソースが足りないことと、CSRF の脅威が Nodejs で使われるようなアプリケーション構成では無視できる場合が多いことを理由に、現在ではメンテナンスされていません。

> The Express.js project does not have the resources to put into this module, which is largely unnecessary for modern SPA-based applications.
> https://github.com/expressjs/csurf

代わりに、サードパーティーが開発しているミドルウェアを、npm という Node.js のパッケージリポジトリで適宜探して使うように指示しています。
https://www.npmjs.com/search?q=express%20csrf
