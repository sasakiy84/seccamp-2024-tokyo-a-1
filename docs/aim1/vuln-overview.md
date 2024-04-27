# ウェブページ閲覧で生じる可能性のある危険

## ユーザーが持つリソースによる整理
リソースとは、「他者に入手、あるいは利用されると不利益を被る可能性のある物体や情報のこと」とここでは定義します。

ウェブページ閲覧における危険という文脈を考えたときに、以下のような形でリソースを整理してみました。
以下で示す分類は排他的にきれいに分類できるわけではありませんが、整理には役立つでしょう。

### ハードウェアの物理的なリソース
例えば、マイク、カメラ、bluetooth、GPSなどがわかりやすいでしょう。
さらには、端末の記憶装置や演算装置、入出力装置なども含まれます。

このリソースが悪用されると、端末を操作している利用者自身や周囲の環境、物体に直接的に作用したり、利用者自身の現実世界に関する物理的な情報を取得されてしまうことが起こりえます。
例えば、カメラに関する権限管理がなされていない場合、利用者の顔などを攻撃者が自由に送信できてしまうでしょう。

ハードウェアリソースのうちの一部は、ブラウザの Pewerful feature の一部として [Permissions API](https://developer.mozilla.org/ja/docs/Web/API/Permissions) というものにより状態を取得できるようになっています。
ただし、ブラウザエンジンごとに権限の状態を取得できる API の種類は異なるようです。（[Chronium](https://chromium.googlesource.com/chromium/src/+/refs/heads/main/third_party/blink/renderer/modules/permissions/permission_descriptor.idl)、[Firefox](https://searchfox.org/mozilla-central/source/dom/webidl/Permissions.webidl#10)、[Webkit](https://github.com/WebKit/WebKit/blob/main/Source/WebCore/Modules/permissions/PermissionName.idl)）

権限の取得は、現在は Permission API からは行えず、API ごとに決められた方法でユーザーに権限をリクエストするようになっています。
たとえば、Notification API は`Notification.requestPermission()`で権限をリクエストできる（[参考](https://developer.mozilla.org/ja/docs/Web/API/Notification/requestPermission_static)）のに対して、

これらのリソースは、どこまで制限するべきか、あるいはしないべきかといった論点や、どのレイヤで対策をとることができてどのレイヤが対策をとるべきなのかという論点、そしてどのように権限を与えるべきかという論点があります。

### ユーザーデータ
ユーザーに関する情報を持った電子的なデータのことです。

#### ウェブサイトに関わるデータ
大きくクライアントに保管されているデータとサーバーに保管されているデータに分けられるでしょう。
具体的には、パスワード、トークン、その他アプリケーションに関わる情報（メールデータやクレジットカードの情報など）です。

#### ウェブサイトに関わらないデータ
他のアプリに保存されているデータや OS レベルで保持されているデータです。

具体的には、所持しているファイルや、設定されている環境変数、他のアプリに紐づいている外部とのやりとりに必要なデータなどです。

## 脆弱性/攻撃方法による整理
たとえば以下のような整理があります。

- CWE
- 安全なウェブサイトの作り方
- MITRE ATT&CK
- OWASP Attacks list

## 主要な脆弱性/攻撃方法の概要
### XSS
XSS は正式名称を Cross-Site Scripting といい、Web に関するある種の脆弱性、あるいはその脆弱性をついた攻撃手法のことです。
インジェクション（注入）攻撃の一種であり、悪意のあるコードが正規のウェブサイトに作成者の意図しない形で注入されることにより引き起こされます。

今回の主題ではないので、簡単な説明にとどめます。

### SQLi
正式名称は SQL Injection といい、インジェクション攻撃の一種で、サーバー側で実行される SQL に不正な文字列が注入されてしまう脆弱性、あるいはそれを用いた攻撃のことをいいます。

### DoS
正式名称は Denial of Service といい、サービスを提供するコンピューターやそれが接続するネットワークを何らかの方法でアクセスできなくする攻撃のことです。

### CSRF
CSRF は、正式名称を Cross-Site Request Forgery といい、Web に関するある種の脆弱性、あるいはその脆弱性をついた攻撃手法のことです。略語は、シーサーフと読むらしいです。
脆弱性を説明するときによく参照される CWE の説明を元に、CSRF とはなにかを説明します。

CWE（Common Weakness Enumeration）とは、共通脆弱性タイプ一覧と訳され、「ソフトウェアにおけるセキュリティ上の弱点（脆弱性）の種類を識別するための共通の基準を目指して」策定されたものです。

https://www.ipa.go.jp/security/vuln/CWE.html

英語版

> The web application does not, or can not, sufficiently verify whether a well-formed, valid, consistent request was intentionally provided by the user who submitted the request.
> https://cwe.mitre.org/data/definitions/352.html

日本語版

> 本脆弱性が存在する Web アプリケーションは、フォーマットに沿った、妥当で一貫性のあるリクエストが、送信したユーザの意図通りに渡されたものかを十分に検証しない、あるいは検証が不可能です。
> https://jvndb.jvn.jp/ja/cwe/CWE-352.html


つまり、概要としては、「ウェブアプリケーションが、リクエストがユーザーが渡したものかどうかを検証しない / できない」脆弱性が CSRF に該当します。

詳細説明の項目も確認しておきましょう

英語版

> When a web server is designed to receive a request from a client without any mechanism for verifying that it was intentionally sent, then it might be possible for an attacker to trick a client into making an unintentional request to the web server which will be treated as an authentic request. This can be done via a URL, image load, XMLHttpRequest, etc. and can result in exposure of data or unintended code execution.
> https://cwe.mitre.org/data/definitions/352.html#Extended_Description

日本語版

> Web サーバがリクエストを検証せずに受け取るよう設計されている場合、攻撃者がクライアントを騙し、意図しないリクエストを Web サーバに送信させる可能性があります。その場合、Web サーバはそのリクエストを正規のものとして取り扱います。
> この攻撃は URL、画像の読み込み、XMLHttpRequest 等を介して行われ、データの漏えいや意図しないコードの実行を招く可能性があります。
> https://jvndb.jvn.jp/ja/cwe/CWE-352.html

ここで注目したいのは、「it might be possible for an attacker to trick a client into making an unintentional request to the web server」の部分です。日本語では「攻撃者がクライアントを騙し、意図しないリクエストを Web サーバに送信させる可能性があります」と訳されています。

新しい情報として、クライアントを騙すという条件が追加されています。ここでいうクライアントとは、被害者のクライアント端末のことだと理解してよいでしょう。これは、SQL Injection のように攻撃者の PC からリクエストを送るのではなく、一般人である被害者がリクエストを送るということを表しています。

