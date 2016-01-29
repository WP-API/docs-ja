---
page: introduction
title: WP REST API version 2.0 Introduction
include_title: No
---

<div class="hero">

	<img src="{{ site.baseurl }}assets/images/banner.jpg" class="banner" />

	<p class="subtitle">HTTP REST APIでWordPressを自由自在！</p>

	<a href="https://wordpress.org/plugins/rest-api/" class="download button radius">
		ダウンロード
		<span>(Version 2.0 beta 11, for WordPress 4.4.1+)</span>
	</a>

	<p class="status">
		<a href="https://github.com/WP-API/WP-API">View on GitHub</a><br />
		<a href="https://travis-ci.org/WP-API/WP-API">
			<img alt="Build Status" src="https://travis-ci.org/WP-API/WP-API.svg?branch=develop" />
		</a>
	</p>

</div>

はじめに
-----

WordPressはアプリケーションフレームワークへと生まれ変わろうとしています。このプロジェクトは、簡単で理解しやすく、信頼性の高いAPIフレームワークを開発することを目的として生まれました。そしてWordPress本体のAPIを開発することも目的としています。

このプラグインは簡単なHTTPベースのREST APIを提供します。あなたのサイトのユーザー、投稿、タクソノミー、その他のデータに対してシンプルなHTTPリクエストを送信することで、取り出したりアップデートすることが可能です。

投稿を取得したい？ `GET` リクエストを `/wp-json/wp/v2/posts` に送るだけです。 IDが4のユーザーをアップデートしたい? `POST` リクエストを `/wp-json/wp/v2/users/4` に送ってください。 "awesome"という単語が含まれるすべての投稿を検索するには、`GET /wp-json/wp/v2/posts?search=awesome` です。 たったこれだけ。簡単ですよね。

このAPIは、WordPressの投稿API、メタデータAPI、ユーザーAPIなどにアクセスするWP Queryのシンプルなインターフェースです。もしあなたがWordPressで何かを始めたいのなら、このAPIがそれを可能にするでしょう。
