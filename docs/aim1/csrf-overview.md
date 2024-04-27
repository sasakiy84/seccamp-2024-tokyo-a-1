# CSRF の概要
## ウェブサービスにおける副作用のある処理
CSRF はウェブサービスの副作用のある処理を利用して攻撃を行います。
副作用のある処理とは、ここではサーバー側の状態を変化させる処理と定義します。
たとえば、データベースを変更する処理や外部決済サービスに決済を依頼する処理などです。


## CSRF の概要
具体的なシナリオに沿って考えてみましょう。

多くのログインが必要なウェブサイトには、パスワードリセット機能があります。実際にパスワードをリセットするときには、「パスワードをリセットするよ～」というリクエストをサーバーに送信するわけです。

このとき、CSRF の脆弱性がある実装方法だった場合、つまり、「パスワード変更のリクエストが正規ユーザーからのリクエストかどうかを検証できないような実装方法だった場合」のことを考えましょう。

アプリケーションは、正規ユーザーかどうか検証できないため、攻撃者のリクエストを受け取るとパスワード変更を実行してしまいます。つまり、リクエスト内に含まれる変更後のパスワードを攻撃者が指定できる場合、そのアカウントの乗っ取りが成功してしまうのです。

もう一つ具体例を出します。

掲示板のようなウェブサービスがあったとします。この掲示板は会員登録が不要で、誰でも送信が可能です。ユーザーの識別には IP アドレスを使っていました。
このとき、攻撃者が CSRF の脆弱性を利用して、被害者の PC から不適切な発言を投稿するリクエストを送信したとします。
この場合、送信元の IP アドレスは被害者のものですから、ほかのユーザーからみると被害者が不適切発言を投稿したように見えます。
このような事例が実際に発生したのが、「パソコン遠隔操作事件」のうちの一つです。
これは CSRF の脆弱性があるウェブサイトに不適切投稿がされた事件で、警察が投稿者の識別に IP アドレスを使った結果、誤認逮捕につながりました。[^4]


## CSRF の被害

脆弱性がある場合、気になるのはどのような被害が発生するか、という点です。
CWE には次のように書かれています

英語版

> Gain Privileges or Assume Identity; Bypass Protection Mechanism; Read Application Data; Modify Application Data; DoS: Crash, Exit, or Restart.
> The consequences will vary depending on the nature of the functionality that is vulnerable to CSRF. An attacker could effectively perform any operations as the victim. If the victim is an administrator or privileged user, the consequences may include obtaining complete control over the web application - deleting or stealing data, uninstalling the product, or using it to launch other attacks against all of the product's users. Because the attacker has the identity of the victim, the scope of CSRF is limited only by the victim's privileges.
> https://cwe.mitre.org/data/definitions/352.html#Common_Consequences

日本語版

> 権限の取得やなりすまし、防御メカニズムの回避、アプリケーションデータの読み取り。重大性は CSRF の脆弱性が存在する機能の性質によって変わります。攻撃者は事実上、被害者と同じように操作を行うことが可能です。被害者が管理者あるいは権限のあるユーザだった場合には、web アプリケーションの完全なコントロール(データの削除や窃取、製品のアンインストールや製品の全てのユーザに対する攻撃の基盤としての利用等)を与えることになります。攻撃者は被害者の識別を持っているため、CSRF の及ぶ範囲は被害者の持つ権限内に制限されます。
> https://jvndb.jvn.jp/ja/cwe/CWE-352.html

重要なのは、脆弱性が存在する機能の性質によって重大性が変わるということです。

たとえば、具体例で見たのは、パスワードリセットの機能が悪用された例でした。この場合、変更できるパスワードが一般ユーザーのものだけであれば、被害は一人のユーザーにとどまりますが、管理者ユーザーのパスワードが変更された場合は、より大きな被害になるでしょう。クレジットカードが使えるかどうかなどでも重大度は変わりそうです。
例では、掲示板に不適切投稿される例もみました。こちらはパスワードの変更に比べれば重大度は低いといえそうです。もちろん脆弱性には変わりませんし、場合によっては生じる不利益が大きくなることもあり得ます。

逆に、パスワード変更の方式がもう少し慎重なものだった場合を考えてみましょう。たとえば、パスワードが変更される前にメールでユーザーに確認が行く場合は、CSRF の脆弱性はありつつも、そこまで重大性は大きくないでしょう。

## CSRF の位置づけ
### CWE
英語：https://cwe.mitre.org/data/definitions/352.html#Common_Consequences
日本語：https://jvndb.jvn.jp/ja/cwe/CWE-352.html

CVE において、CSRF は "Insufficient Verification of Data Authenticity" (不適切な入力確認) に分類されるとしています。
また、XSS (クロスサイトスクリプティング) により CSRF が発生する可能性にも言及されています。


### 安全なウェブサイトの作り方
https://www.ipa.go.jp/security/vuln/websecurity/ug65p900000196e2-att/000017316.pdf#page=32

『安全なウェブサイトの作り方』は、IPA が公開する資料です。
「IPA が届出を受けた脆弱性関連情報を基に、届出件数の多かった脆弱性や攻撃による影響度が大きい脆弱性を取り上げ、ウェブサイト開発者や運営者が適切なセキュリティを考慮したウェブサイトを作成」できるようにすることを目的としています。
https://www.ipa.go.jp/security/vuln/websecurity.html

この資料では、ウェブアプリケーションに関わる 11 種類の主要な脆弱性が挙げられているのですが、そのなかの一つとして「CSRF」が入っています。
そこでは、発生しうる脅威として以下の例が挙げられています。

- ログイン後の利用者のみが利用可能なサービスの悪用
- ログイン後の利用者のみが編集可能な情報の改ざん、新規登録
また、根本的解決として、以下の手法が挙げられています

- 処理を実行するページを POST メソッドでアクセスするようにし、その「hidden パラメータ」に秘密情報が挿入されるよう、前のページを自動生成して、実行ページではその値が正しい場合のみ処理を実行する。
- 処理を実行する直前のページで再度パスワードの入力を求め、実行ページでは、再度入力されたパスワードが正しい場合のみ処理を実行する。
- Referer が正しいリンク元かを確認し、正しい場合のみ処理を実行する。[^1]

### OWASP TOP 10
ウェブアプリケーションについてのセキュリティリスクをランキング化したものです。
おおよそ 4 年ごとに改訂されています。

CSRF は、 2010 年では 5 位[^2]、2013 年では 8 位[^3]にランクインしています。


これが、2017 年版ではランキング外となっています。
ランキング外となった理由については、以下のように述べられています。
> many frameworks include CSRF defenses, it was found in only 5% of applications
> https://owasp.org/www-project-top-ten/2017/Release_Notes


> 多くのフレームワークがこの対策を講じており (CSRF対策)、アプリケーションの5%程度でのみ観察されています。
> https://wiki.owasp.org/images/2/23/OWASP_Top_10-2017%28ja%29.pdf

2024 年 4 月時点の最新版である 2021 年版を確認してみると、トップレベルの項目で CSRF についての記載はないのですが、ランキング 1 位の Broken Access Control の中に CSRF が含まれています。[^4]
[^4]: https://owasp.org/Top10/A01_2021-Broken_Access_Control/

### MITRE ATT&CK
https://attack.mitre.org/

MITRE ATT&CK 　(Adversarial Tactics, Techniques, and Common Knowledge) は、「脆弱性を悪用した実際の攻撃を戦術と技術または手法の観点で分類したナレッジベース」です。
https://www.intellilink.co.jp/column/security/2020/060200.aspx

ここでは、直接 CSRF に言及されてはいないですが、CSRF で発生するような事象が攻撃の全体のどこに位置づけられるかを調べることができます。
たとえば、"Defense Evasion (防御回避) > Use Alternate Authentication Material (代替された認証情報の使用) > Web Session Cookie" という項目があります。
そこでは、Web Session Cookie のテクニックを使うことで、"an adversary can access sensitive information, read email, or perform actions that the victim account has permissions to perform." (攻撃者は、機密情報へのアクセスやメールの閲覧、被害者のアカウントで実行可能な操作などができる) とされています。
https://attack.mitre.org/techniques/T1550/004/

CSRF は、攻撃者が直接的に cookie を入手できるわけではありませんが、cookie を付与したリクエストを送ることによる副作用のある操作はされてしまいます。
ここでは、CSRF が防御の回避、とくにパスワード認証や二段階認証の回避などに使われることが確認できました。

また、"Execution > User Execution > Malicious Link" では、ユーザーが悪意のあるリンクをクリックすることによって、ユーザーのクライアント上で悪意のあるコードを実行させる方法が示されています。
このような悪意のあるリンクを踏ませるというテクニックは、CSRF を実行させるためのテクニックとして使われることが多いです。
このような攻撃手法への対策としては、"User Training" などが、検知手法としては、ネットワークトラフィックの監視と解析などが挙げられています
https://attack.mitre.org/techniques/T1204/001/


---


[^1]: 現在では、Referer ヘッダーではなく、Origin ヘッダーを使うほうがよいです
[^2]: https://owasp.org/www-pdf-archive/OWASP_Top_10_-_2010.pdf
[^3]: https://owasp.org/www-pdf-archive/OWASP_Top_10_-_2013.pdf
[^4]: https://takagi-hiromitsu.jp/misc/misidentification2012/kanagawa.pdf
