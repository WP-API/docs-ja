---
page: introduction
title: APIリファレンス
has_superbar: Yes
---

{% capture intro %}
# APIリファレンス

APIは[REST][]にのっとって整備されています。私達のAPIは、予測しやすく、リソース志向のURLを持ち、APIエラーを示すためにHTTPレスポンスコードを利用します。また、ありきたりなHTTPクライアントでも理解できる、HTTP認証やHTTPメソッドのようなHTTPの組み込み機能を利用しています。そして、クロス・オリジンなリソース共有もサポートしており、あなたは私達が提供するAPIでクライアントサイドWebアプリケーションから安全にAPIを利用できます。すべてのAPIレスポンスはJSONで返却され、エラーもそこに含まれます。

[REST]: https://ja.wikipedia.org/wiki/REST
{% endcapture %}

<section class="route">
	<div class="primary">{{ intro | markdownify }}</div>
</section>

