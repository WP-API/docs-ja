---
title: 探索
---

クライアントが未知のサイトにアクセスする際、
そのサイトでは何が可能でどのように設定されているのかを知る必要があります。
幾つかのステップで探索することができます。


API の探索
-------------------

あるウェブサイトに接続しようとするときの最初のステップは、対象のサイトで API が有効化されているかどうかを確かめることです。
Typically, you'll be working with URLs from user input, so the site you're accessing could be anything. 
探索の手順を踏むことで、API が利用可能であるかを確かめ、アクセス方法を知ることができます。

### リンクヘッダー

探索を始めるのに好ましい手法は、URL に対して HEAD リクエストを送信することです。
REST API はフロントエンドのすべてのページに自動的に以下のような LINK ヘッダーを追記します。

```
Link: <http://example.com/wp-json/>; rel="https://api.w.org/"
```

この URL は API の root route （ルーティングの根っこ）となる `/` を指し示しており、
これを次の探索ステップで使っていきます。

Pretty Permalinks が有効化されていないサイトでは、
WordPress が `/wp-json/` を扱うことができません。従って、
WordPress のデフォルトのパーマリンクが代わりに使われ、リンクヘッダーは以下のようになります。

```
Link: <http://example.com/?rest_route=/>; rel="https://api.w.org/"
```

クライアントを作るときには、このバリエーションがあることを念頭に置き、
どちらのルート(route)であっても動くようにすべきです。

この探索ステップは WordPress が動いているすべてのサイトのすべて URL に対して有効な飲んで、
ユーザーの入力を事前に編集する必要はありません。
HEAD リクエストなので盲目的にいろんなサーバにリクエストを送っても副作用を心配する必要もありません。

<div class="note warning">
{% capture note %}
注: `rel` 要素は WordPress 4.4 以降では `https://api.w.org/` となります。
本プラグインの開発期間中は、`https://github.com/WP-API/WP-API` となっていました。
新しくコードを書くときには新しい方だけを利用してください。
{% endcapture %}

{{ note | markdownify }}
</div>

### `<link>` 要素

HTML をパースできるクライアント、あるいはブラウザの場合、
LINK ヘッダーと同じように利用できるのは html の `<head>` に含まれる `<link>` 要素となります。

```html
<link rel='https://api.w.org/' href='http://example.com/wp-json/' />
```

ブラウザで動いている Javascript であれば DOM から取得できます。

```js
// jQuery method
var $link = jQuery( 'link[rel="https://api.w.org/"]' );
var api_root = $link.attr( 'href' );

// Native method
var links = document.getElementsByTagName('link');
var link = Array.prototype.filter.apply( links, function ( item ) {
	return ( item.rel === "https://api.w.org/" );
} );
var api_root = link[0].href;
```

ブラウザ内のクライアント、あるいは HTTP ヘッダーにアクセス出来ないクライアントでは
この方法のほうが API 探索が容易と思われます。
Atom / RSS フィードの発見方法に似てますので、既存のコードを利用したらいいと思います。

### RSD (Really Simple Discovery)

For clients with support for XML-RPC discovery, the [RSD method][] may be more
applicable. This is an XML-based discovery format typically used for XML-RPC.
RSD has two steps. The first step is to find the RSD endpoint, supplied as a
`<link>` element:

```html
<link rel="EditURI" type="application/rsd+xml" title="RSD" href="http://example.com/xmlrpc.php?rsd" />
```

[RSD method]: http://cyber.law.harvard.edu/blogs/gems/tech/rsd.html

The second step is to fetch the RSD document and parse the available
endpoints. This involves using an XML parser on a document like the following:

```xml
<?xml version="1.0" encoding="utf-8"?>
<rsd version="1.0" xmlns="http://archipelago.phrasewise.com/rsd">
  <service>
    <engineName>WordPress</engineName>
    <engineLink>http://wordpress.org/</engineLink>
    <homePageLink>http://example.com/</homePageLink>
    <apis>
      <api name="WordPress" blogID="1" preferred="true" apiLink="http://example.com/xmlrpc.php" />
      <!-- ... -->
      <api name="WP-API" blogID="1" preferred="false" apiLink="http://example.com/wp-json/" />
    </apis>
  </service>
</rsd>
```

The REST API always has a `name` attribute with the value equal to `WP-API`.

RSD is the least-preferred method of autodiscovery for a couple of reasons.
The first step of RSD-based discovery involves parsing the HTML to first find
the RSD document itself, which is equivalent to the `<link>` Element
autodiscovery. It then involves another step to parse the RSD document, which
requires a full XML parser.

Where possible, we suggest avoiding RSD-based discovery due to the complexity,
but existing XML-RPC clients may prefer to use this method if they already
have an RSD parser enabled. For XML-RPC clients which wish to use the REST API
as a progressive enhancement to the codebase, this avoids needing to support
different forms of discovery.


認証の探索
------------------------

認証方法を探してみましょう。
API のルート(root)のレスポンスは API 自身について記述するオブジェクトであり、
`authentication` キーを含んでいます。

```json
{
	"name": "Example WordPress Site",
	"description": "YOLO",
	"routes": { ... },
	"authentication": {
		"oauth1": {
			"request": "http://example.com/oauth/request",
			"authorize": "http://example.com/oauth/authorize",
			"access": "http://example.com/oauth/access",
			"version": "0.1"
		}
	}
}
```

この `authentication` の値は、利用可能な認証メソッドの ID の map （連想配列）になっています。
The options available here are specific to the authentication method itself. 
認証についての詳しい方法は、 [authentication ドキュメント][] のページヘ進んでください。

[authentication ドキュメント]: /guide/authentication/

Extension Discovery
-------------------

API があることを発見したら、次は API が何をサポートしているのかを見てみましょう。
API のインデックスは、 `namespace` という項目を開示しており、API がサポートしているエクステンションがリストされています。

普通の WordPress（ver.4.4）のサイトですと、API のインフラ部分のみが利用可能で API の全エンドポイントは利用できません。oEmbedのエンドポイントは含まれます。

```json
{
	"name": "Example WordPress Site",
	"namespaces": [
		"oembed/1.0/"
	]
}
```

全 API が利用できるサイト（REST API プラグインが有効化されてるなど）であれば、wp/v2 という項目が `namespaces` に含まれます。

```json
{
	"name": "Example WordPress Site",
	"namespaces": [
		"wp/v2",
		"oembed/1.0/"
	]
}
```

これら、コアエンドポイントを利用する前に、API がサポートされているかを `wp/v2` をチェックすることで確認すべきです。WordPress 4.4 は API のインフラ部分は有効化しますが、 wp/v2 配下のコアエンドポイント群は**含みません**。

同じメカニズムは、REST API をサポートしているプラグインを検知するのにも利用できます。たとえば以下のようなルート(routeの方)を登録しているプラグインがあったとしましょう。

```php
<?php
register_rest_route( 'testplugin/v1', '/testroute', array( /* ... */ ) );
```

そうすると、 `testplugin/v1` というネームスペースがインデックスに追加されます。

```json
{
	"name": "Example WordPress Site",
	"namespaces": [
		"wp/v2",
		"oembed/1.0/",
		"testplugin/v1"
	]
}
```
