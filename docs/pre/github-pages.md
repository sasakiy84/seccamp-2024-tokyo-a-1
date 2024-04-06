# GitHub Pages でページをデプロイしてみる

nodejs + GitHub Pages を使って、ネット上に自分のサイトを公開してみます。
これによって、現代の開発における一連のプロセスの概要を体験します。

## GitHub のアカウントを作る
GitHub のアカウントを持っていない人は、[GitHub でのアカウントの作成 - GitHub Docs](https://docs.github.com/ja/get-started/start-your-journey/creating-an-account-on-github)を参考にしながら、アカウントを作成してください。

## Honkit
ウェブサイトを公開するには、以下の三つが必要です。

- 表示するコンテンツ（たとえば HTML）
- サーバー（コンテンツを置いておく場所）
- ドメイン（サーバーの住所）

このうち、サーバーとドメインは GitHub が提供してくれるものを利用します。
コンテンツについては、これから Honkit と呼ばれる Node エコシステムのライブラリを使って作成します。

[About this documentation · HonKit Documentation](https://honkit.netlify.app/)

Honkit は、形の整ったきれいなサイトを簡単に作れるライブラリです。
このドキュメントも Honkit を使って作成されています。

まずは、今回作業するディレクトリを作成しましょう。
コマンドラインか、ファイル操作ツール（Explorer や Finder）を使って、好きな場所に `seccamp-tokyo-a-1-honkit`というフォルダを作成してください。フォルダ名が異なっても構いません。

その後、コマンドライン上でそのディレクトリまで移動し、以下のコマンドを打ってください。
```sh
npm init
git init
```

すると、`.git`フォルダや`package.json`などのファイルが作成されたと思います。

npm は、 node で使えるパッケージの管理ツールです。npm コマンドを使って、ほしいパッケージをインストールできます。
git は、ソースコードのバージョン管理ツールです。このドキュメントでは深い理解は必要ないので、「ソースコードを管理してくれる君」と思っておいてください。

これらのコマンドは、現在のフォルダを 「npm / git で管理します」という設定を行うコマンドです。

次に、`.gitignore`ファイルを作成し、以下の内容をコピペしてください。
```
node_modules/
_book/
```




## GitHub にプッシュする
## GitHub Pages の設定をする
## GitHub Actions を整える
