# ウェブページ閲覧で生じる可能性のある危険

## ユーザーが持つリソースによる整理
リソースとは、他者に入手、あるいは利用されると不利益を被る可能性のあるもののこと。
### ハードウェアの物理的なリソース
スピーカーなども含む
permissions というモデルで管理している。

### ユーザーデータ
#### ウェブサイトに関わるデータ
- パスワード
- クレカ
- クライアントに保管されているデータ
- サーバーに保管されているデータ
#### ウェブサイトに関わらないデータ
- 端末にあるファイル

### ウェブサイトへのアクセス

## 脆弱性/攻撃方法による整理
### CWE
### 安全なウェブサイトの作り方
### MITRE ATT&CK
### OWASP Attacks list

## 主要な脆弱性/攻撃方法の概要
### XSS
### CSRF
CSRF は、正式名称を Cross-Site Request Forgery といい、Web に関する脆弱性、あるいはその脆弱性をついた攻撃手法のことです。略語は、シーサーフと読むらしいです。
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

新しい情報として、クライアントを騙すという条件が追加されています。ここでいうクライアントとは、被害者のクライアントのことです。これは、SQL Injection のように攻撃者の PC からリクエストを送るのではなく、一般人である被害者がリクエストを送るということを表しています。

### SQLi
### DoS
