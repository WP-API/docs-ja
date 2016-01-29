---
title: 認証
---

このAPIにおいては、認証には2つの選択肢があり、基本的に以下のように選びましょう。

* そのサイトで有効化されたテーマやプラグインから利用するのであれば **クッキー認証**
* デスクトップアプリ、ウェブアプリ、モバイルアプリなどのサイトの外からアクセスするクライアントから API を利用するのであれば **OAuth 認証**

クッキー認証
---------------------
クッキー認証は、WordPress に実装された基礎的な認証です。
管理画面にログインするときには、このクッキー認証が利用されており、ログインするユーザーのクッキーが正しく発行されています。
したがって、プラグインやテーマ開発者が必要とするのはログインしているユーザーのみとなります。

しかしながら、REST API では [CSRF][] の問題を解決するために [nonces][] と呼ばれる技術を利用しています。
そのおかげで、他のサイトを経由して、あなたが明示的に意図していない行動をとってしまうのを防いでいるのですが、
同時に、API 利用のために少し特殊な操作が必要になります。

プラグインやテーマの作者がビルトイン Javascript API を利用する場合には、この処理は自動的に行われます（推奨）。
独自のデータモデルからアクセスする場合には、 `wp.api.models.Base` を継承することで、
あらゆるカスタムリクエストで nonce がきちんと送信されることが保証されます。

マニュアルで Ajax リクエストを送る場合、nonce は各リクエストで毎回送信する必要があります。 
送信された nonce は `wp_rest` にセットされたアクションと一緒に利用されます。
API への nonce の受け渡しは、POST のデータや GET のクエリの `_wpnonce` パラメータ、
あるいは`X-WP-Nonce` ヘッダーにセットして行います。

Note: Until recently, most software had spotty support for `DELETE` requests. For instance, PHP doesn't transform the request body of a `DELETE` request into a super global. 従って、ヘッダーとして nonce を供給するやり方が最も信頼できるアプローチとなります。

この認証は WordPress のクッキーに依存しているということは覚えておきましょう。
このクッキー認証が利用できるのは、REST API が WordPress の内部から使われていて、
かつ、ユーザーがログインしている時のみであるということです。
さらに、そのログインユーザーが、実行しようとしているアクションに必要な権限を与えられているということも条件となります。

以下は、ビルトインの Javascript クライアントが nonce を生成する例です。

```php
<?php
wp_localize_script( 'wp-api', 'WP_API_Settings', array( 'root' => esc_url_raw( rest_url() ), 'nonce' => wp_create_nonce( 'wp_rest' ) ) );
```

This is then used in the base model:

```javascript
options.beforeSend = function(xhr) {
	xhr.setRequestHeader('X-WP-Nonce', WP_API_Settings.nonce);

	if (beforeSend) {
		return beforeSend.apply(this, arguments);
	}
};
```

以下は、jQuery AJAX を使って投稿のタイトルを書き換える例です。

```javascript
$.ajax( {
    url: WP_API_Settings.root + 'wp/v2/posts/1',
    method: 'POST',
    beforeSend: function ( xhr ) {
        xhr.setRequestHeader( 'X-WP-Nonce', WP_API_Settings.nonce );
    },
    data:{
        'title' : 'Hello Moon'
    }
} ).done( function ( response ) {
    console.log( response );
} );
```

[nonces]: http://codex.wordpress.org/WordPress_Nonces
[CSRF]: http://en.wikipedia.org/wiki/Cross-site_request_forgery


OAuth 認証
--------------------
OAuth 認証は外部クライアントのための認証方法です。
OAuth 認証でも、ユーザーは通常の WordPress ログイン画面を通じてのみログインを行い、
その後、自身に代わって処理を行うクライアントを認証します。
クライアントには、OAuth トークンが発行され、API へのアクセスが許可されます。このアクセス権はユーザー側からいつでも取り消すことができます。

OAuth 認証は、[OAuth 1.0a specification][oauth] (published as
RFC5849) を利用しており、[OAuth plugin][oauth-plugin] がサイトで
有効化されていることが必要です（このプラグインは将来的にコアにマージされます）。

WP API と OAuth サーバー用のプラグインを有効化したら、コンシューマを作成しましょう。
コンシューマは、アプリケーションの識別子であり、サイトとリンクするために必要な key と secret からなります。

コンシューマの作成には、サーバで以下を実行します。

```bash
$ wp oauth1 add

ID: 4
Key: sDc51JgH2mFu
Secret: LnUdIsyhPFnURkatekRIAUfYV7nmP4iF3AVxkS5PRHPXxgOW
```

上記の key と secret を認証のプロセス全体で利用します。
現状 UI は無いのですが、将来的に導入されます。

利用方法の事例として、[CLI client][client-cli] および、 [API console][api-console] を見てみるといいでしょう。

[oauth]: http://tools.ietf.org/html/rfc5849
[oauth-plugin]: https://github.com/WP-API/OAuth1
[client-cli]: https://github.com/WP-API/client-cli
[api-console]: https://github.com/WP-API/api-console


ベーシック認証
--------------------
ベーシック認証は、外部クライアントから利用できる認証の代替的な方法です。
OAuth 認証は複雑なので、開発段階ではベーシック認証が便利でしょう。
ただし、ベーシック認証ではすべてのリクエストにおいてユーザー名とパスワードを送る必要があり、
本番の環境で利用することは超非推奨です。

ベーシック認証は、 [HTTP Basic Authentication][http-basic] (published as RFC2617) を利用しており [Basic Auth plugin][basic-auth-plugin] プラグインの有効化が必要です。

ベーシック認証の利用時、ユーザー名とパスワードは各リクエストの `Authorization` ヘッダーを通じて渡します。
この値は、HTTP Basic 認証の仕様に従い、base64 でエンコードされるべきです。

以下は、ベーシック認証による、WordPress のHTTP API を使った投稿の更新の例です。

```php
$headers = array (
	'Authorization' => 'Basic ' . base64_encode( 'admin' . ':' . '12345' ),
);
$url = rest_url( 'wp/v2/posts/1' );

$body = array(
	'title' => 'Hello Gaia' 
);

$response = wp_remote_post( $url, array (
    'method'  => 'POST',
    'headers' => $headers,
    'body'    =>  $data
) );
```
    

[http-basic]: https://tools.ietf.org/html/rfc2617
[basic-auth-plugin]: https://github.com/WP-API/Basic-Auth
