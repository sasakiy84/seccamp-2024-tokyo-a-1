# ハンズオン　3-3

この節では、今までの防御手法と少し視点を変えたものを紹介します。
今までの防御手法は、サーバー側でどのようにリクエストを検証するか、という考え方でした。一方で、これから紹介する方法は、クライアントから余計なものを送らないという考え方です。

CSRF の多くは、cookie が送信先のサイトに自動で送信されることを利用しています。逆に考えると、cookie がクロスサイトのリクエストで送信されなければ、通常の認証機構で「未ログイン」としてはじくことができます。

## SameSite 属性

SameSite 属性という cookie のオプションがあります。これを使えば、送信元と送信先のドメインが異なる場合にも自動で付与される cookie に、ドメインが異なる場合に送信しないという設定を加えることができます。
https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Set-Cookie/SameSite

SameSite 属性は、`Lax`, `Strict`, `None`の三つの値をとりますが、このうち`Lax`, `Strict`を使うと、異なるオリジンへのリクエストに cookie を送信しなくなります。
`Lax`, `Strict`の違いは、Top Level Navigation のときに cookie を送信するかです。トップレベルナビゲーションとは、`a`タグを使ったリンク遷移などのことです。`Lax`に設定されているときは、Top Level Navigation のときは cookie を送信します。

https://datatracker.ietf.org/doc/html/draft-ietf-httpbis-rfc6265bis-05#section-8.8.2

Top Level Navigation を特別対応にするのは、ユーザーの利便性のためです。

## 挙動確認

以下のエンドポイントを用意しました。

cookie をセットする
http://localhost:3000/5-samesite-attr

このリンクにアクセスすることで、以下の cookie をセットしろ、というレスポンスが返ります。

- samesite-default: samesite-default
- samesite-lax: samesite-lax
- samesite-strict: samesite-strict
- samesite-none: samesite-none
- samesite-none-unsecure: samesite-none-unsecure

それぞれ、対応した属性がセットされています。
次に、検証画面の Application タブから cookie を見てみましょう。おそらく、samesite-none-unsecure の cookie はセットされていないはずです。
これは、ブラウザが Secure 属性がなく、 `SameSite: none`の cookie を許可していないためです。ブラウザのエラーにもその旨が表示されています。

> Mark cross-site cookies as Secure to allow setting them in cross-site contexts

![](img/guard-cookie_20221013235149.png)

次に、表示されているリンクをクリックしてみましょう。このリンクは SameSite での遷移ですから、もともとセットされていない samesite-none-unsecure 以外の cookie はきちんと取得されているはずです。

次に、このページの以下のリンクから、先ほどと同じページに遷移してみましょう。
http://localhost:3000/5-check-cookies

samesite-strict が undefined になったと思います。クロスサイトでの遷移ですから、予想した通りですね。一方で、lax は相変わらず取得できていることもわかると思います。なお、samesite-default は、chrome の場合は lax と同じ挙動をするはずです。

最後に、POST を試してみましょう。以下のボタンをクリックすると、http://localhost:3000/5-check-cookies に POST リクエストが送られます。

<div>
<form id="form" action="http://localhost:3000/5-check-cookies" method="post">
  <button type="submit">submit</buttion>
</form>
</div>

samesite-none のみが取得されていると思います。

## 制約

当然ですが、cookie を使った認証処理をしているサイトでしか使えません。

## Origin Header の確認
ブラウザの機能として、Origin Header というものがあります。

以下のリンク先のページの Submit を実行すると、サーバー側にログが流れると思います。
http://localhost:3000/4-prepare-custom-header

## 今までの防御手法の確認
Chrome では　バージョン 80 以降で cookie の samesite 属性をデフォルトで `lax` にしています。
お気づきの人もいると思いますが、いままでのデモでは`samesite=none`を強制的に指定していました。

この指定を外すと、今までのデモはどうなるでしょうか？
特に CSRF token を使っていない POST の例について、どのような挙動をするか試してみてください。


