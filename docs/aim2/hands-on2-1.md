# ハンズオン 2-1

## HTTP メソッド

GET や POST といった HTTP メソッドについて概要を説明します。

HTTP メソッドは、指定した URL にどのような操作を要求するのかを、クライアントがサーバー側に伝えるものです。
たとえば、GET メソッドを使った場合は、指定した URL のリソースを取得すること求めていますし、POST メソッドを使った場合は、指定した URL に対して、なんらかの情報を送信して、それに紐づいた操作を行うことを求めます。

詳しくは下記の記事を参考にしてください

HTTP メッセージ - HTTP | MDN
https://developer.mozilla.org/ja/docs/Web/HTTP/Messages

HTTP リクエストメソッド - HTTP | MDN
https://developer.mozilla.org/ja/docs/Web/HTTP/Methods

重要なことは、サーバー側のアプリケーションにとって HTTP メソッドはただの取り決めであり、求められた理念とは異なる処理を容易に実行できてしまうことです。
理念とは異なる処理とは、例えば GET メソッドにパスワード変更の処理を紐づけるなどです。

## サンプルアプリの説明
サンプルアプリは以下のような実装です

```ts
// /3-post-password-change-dangerous というパスに post メソッドのリクエストがきたときにこの処理を実行する
router.post("/3-post-password-change-dangerous", (req, res) => {
  // cookie から sessionId をとりだし、それを検証してログイン済みかどうか確かめる
  const sessionId = req.cookies["sessionId"];
  const { newPassword } = req.body;
  if (!sessionId || !sessionIds[sessionId]) res.send("login required");
  //   新しいパスワードが送られてきているか確かめて、パスワードを更新する
  else if (typeof newPassword !== "string") res.send("newPassword is required");
  else {
    const userName = sessionIds[sessionId].userName;
    changePassword(userName, newPassword);
    res.send("password changed!");
  }
});
```

前節のコードと大きく変わったところは、HTTP メソッドが get から post になったことです。
では、このエンドポイントをどのように使うのかを確認しましょう。
## パスワード変更の正常系

ユーザーは、以下のページにアクセスします。

http://localhost:3000/3-post-password-change-dangerous

すると、フォームが表示されます。ユーザーは新しいパスワードを入力し、submit ボタンを押します。
すると、http://localhost:3000/3-post-password-change-dangerous に post リクエストがとび、パスワードが変更されます。
実際にパスワードを変更してみましょう。サーバー側にログが出ていることを確認できるはずです。
なお、get リクエストのときは、以下のような処理が実行されています

```ts
router.get("/3-post-password-change-dangerous", (_req, res) => {
  res.send(`
    <form id="form" action="http://localhost:3000/3-post-password-change-dangerous" method="post">
    <input name="newPassword" type="text" value=""></input>
    <button type="submit">submit</buttion>
    </form>
    `);
});
```

## 防御の検証

POST メソッドを使ったことで、前回と同じような攻撃はできなくなっています。その理由を以下に説明します。

前回は、悪意のあるリンクをユーザーに送っていました。そして、ユーザーがそのリンクを踏むと同時にパスワード変更のリクエストが送信され、ユーザーのパスワードが攻撃者の指定したパスワードに変更されてしまいました。
POST メソッドの場合を考えてみると、以下の二つの点から上記の攻撃ができなくなっていることがわかります。

- リンクをクリックしたときには GET リクエストが送られる。POST リクエストを送信するためには、フォームを使ってユーザーが明示的に submit ボタンを押さなければならないこと
- URL クエリパラメータで攻撃者が新しいパスワードを指定することができないこと

そのため、たとえば攻撃者が以下のようなリンクをユーザーに送ったとしても、前節のように攻撃が成功することはありません。
http://localhost:3000/3-post-password-change-dangero?newPassword=evil

## 攻撃の検証

しかし、この post メソッドを使った防御はあっさりと破られます。
順番に見ていきましょう。

### URL クエリパラメータで攻撃者が新しいパスワードを指定することができないこと

まず、「URL クエリパラメータで攻撃者が新しいパスワードを指定することができないこと」について攻撃側の対応策を考えます。

ようは、フォームの初期値を指定できればいいのです。そして、なんとフォームの input タグは初期値を指定できます。
そのため、自分でウェブサイトを作って、フォームの初期値を指定したページを用意します。
そして、submit したときの送信先を http://localhost:3000/3-post-password-change-dangerous にすれば、攻撃者が指定したパスワードで更新させることが可能です。
ついでに、新しいパスワードの input 要素を CSS でめっちゃ小さくしたり、透明にしてユーザーから見えなくし、submit ボタンをリンク遷移のボタンなどに見せかければ完璧でしょう。

### POST リクエストを送信するためには、フォームを使ってユーザーが明示的に submit ボタンを押さなければならないこと

次に、「POST リクエストを送信するためには、ユーザーが明示的に submit ボタンを押さなければならないこと」について対応策を考えます。
JavaScript を使うことで、自動でフォームを送信することが可能です。
フォームの初期値設定と組み合わせることで、ユーザーがリンクをクリックしてページが読み込まれた瞬間に、攻撃者が指定したパスワードに更新する POST リクエストを自動で送ることができるのです。

### 具体的な攻撃

攻撃者は、以下のようなページを用意します。

```ts
app.get("/3-post-password-change-dangerous", (_req, res) => {
  res.send(`
  <form id="form" action="${ORIGIN_BASE_URL}/3-post-password-change-dangerous" method="post" style="display: none;">
  <input name="newPassword" type=text value="aaaaaaa"></input>
  <button type="submit">submit</buttion>
  </form>
  <script>
    // form 要素を JavaScript で操作できるように取得する
    const form = document.getElementById("form")
    // form を操作して、submit する
    form.submit()
  </script>
`);
});
```

注目してほしいのは、以下の二点です。
まず、こちらでフォームの初期値を `aaaaaaa` に設定しています。

```html
<input name="newPassword" type=text value="aaaaaaa"></input>
```

次に、こちらでこのスクリプト部分が読み込まれた瞬間にフォームを送信しています。

```ts
// form 要素を JavaScript で操作できるように取得する
const form = document.getElementById("form");
// form を操作して、submit する
form.submit();
```

実際に攻撃してみましょう。以下のリンクにアクセスすると、上記のコードが実行され、パスワードが変更されたというログがでるはずです。
http://127.0.0.1:4000/3-post-password-change-dangerous



## 防御の方針

ここで一度 CWE の定義を思い出してみます。
英語版

> The web application does not, or can not, sufficiently verify whether a well-formed, valid, consistent request was intentionally provided by the user who submitted the request.
> https://jvndb.jvn.jp/ja/cwe/CWE-352.html

日本語版

> 本脆弱性が存在する Web アプリケーションは、フォーマットに沿った、妥当で一貫性のあるリクエストが、送信したユーザの意図通りに渡されたものかを十分に検証しない、あるいは検証が不可能です。
> https://cwe.mitre.org/data/definitions/352.html

CWE の定義によると、CSRF 脆弱性の原因について、ユーザーの意図通りに送信されたものかが「検証できない」ことだとされています。そのため、どうにかして正規のリクエストかどうかを検証することを考えます。

### 攻撃手法の整理

前章までの解説を踏まえて、エンドポイントは POST で実装しているとします。

そのような場合を考えると、攻撃者は一度自分が作成したウェブサイトを経由して攻撃を行う必要があります。
ユーザーに POST メソッドを使ったリクエストを送信させるためには、HTML フォームの submit などを実行する必要があるからです。
また、ユーザーのブラウザ上で悪意のあるコードを実行させる必要があります。
それは、cookie の sessionId を利用するためであったり、IP アドレスを偽装する必要があるからです。

### 防御手法の提案

「攻撃者は一度自分が作成したウェブサイトを経由して攻撃を行う」ことに注目します。

もし、リクエストが正規のサイト（運営者が構築したサイト）から送信されたものか、その他のウェブサイトから送信されたものかをサーバーアプリケーション側で検証できたとしたら、自分のサイトから送られたリクエスト以外を怪しいリクエストとして弾くことが可能なはずです。
言葉を変えれば、正規のサイトから送信されるリクエストに検証可能なもの（非正規のサイトからは送信できないもの）を紐づけることで、リクエストの検証を行えばいいのではないか、ということです。

## 防御の実装

「正規のサイトから送信されるリクエストに検証可能なもの（非正規のサイトからは送信できないもの）を紐づける」という方針で、現在もっとも安全で汎用的に使えるとされている、csrf token を使った実装方法を説明します。

## 実装の方針

GET 陸末ストに対してフォームなどを含んだ HTML を返すときに、csrf token と呼ばれる、ランダムな文字列を生成します。そして、それをサーバーサイドとクライアントサイドに保存します。
次に、クライアントからの POST リクエストに csrf token が含まれるようにして、サーバー側に保存してあったトークンと検証することで、正規のサイトからのリクエストかどうかを判定します。

攻撃者視点で考えてみましょう。
csrf token は、ユーザーがフォームのページに GET リクエストを出したときに毎回生成されるので、事前に知ることは不可能です。そのため、自分が用意する攻撃用のウェブサイトから、有効な csrf token を含んだリクエストを送信することができません。結果として、サーバー側のアプリケーションは、悪意のあるウェブサイトからのリクエストかどうかを判定することができます。

正確には、攻撃者が csrf token を知ることができるかもしれない方法があるのですが、それはブラウザの防御機構によって防がれています。このことについては 同一オリジンポリシーで後述します。

## 実装

上記のような方針を具体的に実装すると、以下のようになります。

```ts
router.get("/3-post-password-change-csrf-token", (_req, res) => {
  // csrf token を生成
  const csrfToken = crypto.randomUUID();
  //   サーバー側に csrf token を保存
  csrfTokens.push(csrfToken);
  //   form に csrf token を埋め込んだ HTML を送信
  res.send(`
    <form id="form" action="http://localhost:3000/3-post-password-change-csrf-token/" method="post">
    <input name="newPassword" type="text" value=""></input>
    <input name="csrfToken" type="hidden" value="${csrfToken}"></input>
    <button type="submit">submit</buttion>
    </form>
    `);
});

router.post("/3-post-password-change-csrf-token", (req, res) => {
  // sessionId を使ってログイン済みかどうかを検証
  const sessionId = req.cookies["sessionId"];
  // csrf token を取り出す
  const { newPassword, csrfToken } = req.body;
  if (!sessionId || !sessionIds[sessionId]) res.send("login required");
  else if (typeof newPassword !== "string" || typeof csrfToken !== "string")
    res.send("newPassword and csrfToken is required");
  // サーバー側に csrf token が保存されているかを検証
  else if (!csrfTokens.includes(csrfToken)) {
    console.log("attack detected!!!");
    res.send("you are attacker!!!");
  } else {
    const userName = sessionIds[sessionId].userName;
    changePassword(userName, newPassword);
    res.send("password changed!");
  }
});
```

二点解説を加えます。

```ts
csrfTokens.push(csrfToken);
```

これは、簡易的に csrf token を単純に配列上に保存しています。
また、このアプリでは一連の処理が終わった後に csrf token を削除していないので、本来であればその処理も必要です。

```html
<input name="csrfToken" type="hidden" value="${csrfToken}"></input>
```

これは、csrf token をフォームに埋め込んでいます。`type="hidden"`となっているため、ユーザーが見ているブラウザには描写されませんが、submit ボタンをおしたときに、新しいパスワードと一緒に送信されます。

## 実際の挙動

実際に動かして確認してみます。下のリンクにアクセスしてみましょう。
http://localhost:3000/3-post-password-change-csrf-token

すると、フォームが表示されるはずです。そのフォームを検証画面で見てみると、以下のように csrf-token が設定してあります。

![](img/guard-csrf-using-token_20221011233146.png)

## 防御の確認

攻撃用のウェブページを用意してみました。実装は以下の通りです。

```ts
app.get("/3-post-password-change-csrf-token", (_req, res) => {
  res.send(`
  <form id="form" action="${ORIGIN_BASE_URL}/3-post-password-change-csrf-token" method="post" style="display: none;">
  <input name="newPassword" type="text" value="aaaaaaa"></input>
  <input name="csrfToken" type="hidden" value="aaaaaaa"></input>
  <button type="submit">submit</buttion>
  </form>
  <script>
      const form = document.getElementById("form")
      form.submit()
  </script>
`);
});
```

しかし、csrf token がわからないため、とりあえず適当な値で埋めています。
このページにアクセスすると、パスワード変更の処理は実行されず、`console.log("attack detected!!!");`が実行されることがわかると思います。
http://localhost:4000/3-post-password-change-csrf-token
