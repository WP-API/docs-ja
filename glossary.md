---
title: 用語集
---

このドキュメンテーションの中でよく使われているフレーズを手っ取り早く理解しましょう。

## コントローラー

[Model-View-Controller][MVC] はソフトウエア開発の標準的な開発手法です。もしあなたがまだ詳しくないのなら、少し勉強しておくとすると先に進むのが速くなると思います。

WP-APIでは、エンドポイントで使用されるクラスの設計で、このコントローラーの考え方を踏襲しており、すべてのエンドポイントは `WP_REST_Controller` クラスを継承し、共通メソッドの信頼性を確保しています。

[MVC]: http://ja.wikipedia.org/wiki/Model-view-controller

## HEAD, GET, POST, PUT, そして DELETE リクエスト

これらの “HTTP verbs (http 動詞)” は、HTTPクライアントが、リソースに対して実行しようとするアクションのタイプを表しています。
たとえば、 `GET` リクエストは投稿のデータを取得するために使われ、 `DELETE` リクエストは投稿を削除するために使われる、というように。
これらの語はまとめて HTTP verbs と呼ばれており、それは web に置いて標準化されたものです。

WordPress の関数に慣れ親しんでいる人であれば、 `GET` は `wp_remote_get()` に、`POST` は `wp_remote_post()` に対応すると言えます。

## HTTP クライアント

“HTTP クライアント”とは、WP-API と関わるためにあなたが利用するツールのことを指します。
ブラウザでは [Postman][] (Chrome) や [REST Easy][] (Firefox)を、コマンドラインでは [httpie][] を使ってリクエストをテストすることができます。

WordPress 自体も HTTP クライアントを提供しており、 
[`WP_HTTP` class][WP_HTTP] とその関連関数群（たとえば `wp_remote_get()` ）などがあり、
ある WordPress から別の WordPress にアクセスするために利用することができます。

[Postman]: https://chrome.google.com/webstore/detail/postman-rest-client/fdmmgilgnpjigdojojpjoooidkmcomcm?hl=en
[REST Easy]: https://github.com/nathan-osman/REST-Easy
[httpie]: https://github.com/jakubroztocil/httpie
[WP_HTTP]: https://codex.wordpress.org/HTTP_API

## リソース

リソースとは、WordPress 内の具体的なエンティティのことです。
Posts, Pages, Comments, Users, Terms などのことで、あなたもすでに知っているものでしょう。
WP-API は、HTTP クライアントからの、こうしたリソースに対する CRUD の実行（Create, Read, Update, Delete）を可能にします。

具体的にどのように WP-API リソースとインタラクトするのかを見てみましょう。

* `GET /wp-json/wp/v2/posts` 
投稿のリストを取得。`WP_Query` に概ね対応する。

* `GET /wp-json/wp/v2/posts/123` 
ID が 123 である投稿の取得。 `get_post()` に対応。

* `POST /wp-json/wp/v2/posts` 
新規投稿。`wp_insert_post()` に対応。

* `DELETE /wp-json/wp/v2/posts/123` 
ID が 123 の投稿を削除。`wp_delete_post()` に対応。

## ルートとエンドポイント

エンドポイントとは、 API を通して利用可能な関数・機能のことで、
たとえば、 API のインデックスを取得したり、投稿を更新したり、
コメントを削除したりということが可能です。
エンドポイントは、いくつかのパラメータを受け付けて特定の関数を実行し、クライアントにデータを返します。

ルートとは、エンドポイントにアクセスするための”名前”のことで、URL の形で利用されます。ひとつのルートは複数のエンドポイントと関連付けられることが可能です。どのエンドポイントに関連付けられるのかは、 HTTP verbs に依って変わります。

`http://example.com/wp-json/wp/v2/posts/123` という URL を例に取ると、

* ルートは `wp/v2/posts/123` となります。- ルートは `wp-json` を含みません。なぜなら、 `wp-json` は API 自身の base path であるからです。

* このルーツには3つのエンドポイントがあります。

  * `GET` は `get_item` メソッドのトリガーとなり、投稿データをクライアントに返します。
  * `PUT` は `update_item` メソッドのトリガーとなり、更新のためのデータを受け取って更新後の投稿データを返します。
  * `DELETE` は `delete_item` メソッドのトリガーとなり、消したばかりの投稿データをクライアントに返します。

**注意:** pretty permalink を有効にしていないサイトでは、
ルートは `rest_route` パラメータとして URL に追加されます。
上記の例で言えば、 `http://example.com/?rest_route=wp/v2/posts/123` がその場合の完全な URL となります。

## スキーマ

スキーマとは、 WP-API のレスポンスデータの形式（フォーマット）を表す言葉です。
たとえば、Post スキーマは、Post が `id`、 `title`、 `content`、 `author`、その他のフィールドを返すようにリクエストを伝達します。
WP-API のスキーマは、各フィールドのタイプを示したり、人間に理解可能なディスクリプションを提供したり、
フィールドが返されるコンテクストを表示したりもします。
