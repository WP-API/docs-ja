---
title: レスポンスを修正する
---
WordPress REST APIのデフォルト・エンドポイントはデータがなにを返すかという観点においてわかりやすい設計になっています。これは80%のサイトとその使用法にあった80/20ルールに則っていますが、これらデータの初期値は数百万とあるサイトすべての要望を満たすものではありません。

REST APIは高度に拡張できる設計になっており、WordPressの<ruby>他の部分<rt>rest</rt></ruby>（ダジャレじゃないですよ）と同様です。この文書では投稿やユーザーのメタ情報に限らないさらなるデータを追加する方法について説明します。`register_rest_field`関数、特定のオプションのレスポンスにフィールドを追加するために設計されたヘルパー関数を使って。

`register_rest_field`はオブジェクトのレスポンスを修正する一つの方法です。たとえば、投稿オブジェクトにフィールドを追加するために使われれば、すべてのレスポンスにおいてそのフィールドが含まれます。一つのオブジェクトか複数のオブジェクトかにかかかわらず、です。

さらにフィールドが追加されたエンドポイントにおいて、その値を更新することもできます。

この文書がおかれた文脈において、「フィールド」という用語はAPIによって返されるオブジェクトのフィールドを意味しているということに注意してください。これは投稿やコメント、ユーザーのメタフィールドについて言っているのではありません。`register_rest_field`はレスポンスにメタフィールドを追加するだけではなく、どんなデータでも追加できるのです。

レスポンスを変更することについての注意書き
---------------------------------------

APIはたくさんのフィールドをAPIレスポンスとして外部にさらします。そこにはあなたが必要としないものや、サイトにふさわしくないものも含まれるでしょう。レスポンスのフィールドを修正したり削除したりすることは、通常のレスポンスを期待しているAPIクライアントに問題を引き起こす**でしょう**。これにはモバイル・クライアントや、サイトを管理する助けとなるサード・パーティのツールも含まれます。

少しのデータしか必要なくても、APIはあらゆるクライアントへのインターフェースとして公開されているのであり、あなたが取り組んでいる機能だけのためのものではないということを念頭においてください。レスポンスを変更することは危険なのです。

フィールドを追加することは危険ではないので、データを修正するよりは、重複していた方がましです。フィールドを削除することは絶対におすすめしません。もっと小さなデータセットが必要ならば、まずはコンテキストを変更することを検討し、独自のコンテキストを作るよう心がけてください。

忘れないで下さい。APIはあなたがレスポンスを変更することを禁止しているのではなく、そうしないようなコードとして設計されているのです。内部的にはフィールドの登録はフィルターによってサポートされており、他の選択肢がまったくない場合は利用できるということです。


`register_rest_field`が行うこと
------------------------------

レスポンスの構造において、グローバル変数$wp_rest_additional_fieldsはフィールドを保持するために使われています。そこにオブジェクトのレスポンスに追加されるオブジェクト名が保持されます。このグローバル変数に追加するためのユーティリティ関数としてREST APIは`register_rest_field`を提供しています。$wp_rest_additional_fieldsに直接追加することは前方互換性の観点から禁止されています。

それぞれのオブジェクト——投稿、ユーザー、ターム、コメント、メタ、その他——のために、$wp_rest_additional_fields はフィールドの配列を保持しており、それぞれが値を取得、更新するためのコールバックを有しています。どのエンドポイントにおいてもそのフィールドが追加されていればこのコールバックを利用できます。

`register_rest_field`の使い方
-------------------------------

関数`register_rest_field`は3つの引数を受け取ります:

1. `$object_type`: オブジェクトの名前。このフィールドが追加されるオブジェクト名の文字列、またはオブジェクト名からなる配列。投稿タイプのエンドポイントに追加する場合は、投稿タイプ名を使用する。他には、"terms", "meta", "user", "comments"などが使われるかもしれません。

2. `$attribute`: フィールド名。レスポンス・オブジェクトのキーとして定義されます。

3. `$args`: コールバック関数を指定するキーを持った配列。コールバックにはデータの取得、フィールドの更新、スキーマの定義があります。それぞれのキーはオプションですが、指定されない場合はこの機能が使えません。

つまり、あなたが値を読み取るコールバック関数を指定したのに更新用のコールバックを指定しなかった場合、読み取りはできますが更新できなくなります。多くのシチュエーションでこうしたことが求められるでしょう。

フィールドは`rest_api_init`で登録されるべきです。`init`ではなく、このアクションを使うことで、REST APIを使わないWordPressリクエストでもフィールドが登録されてしまうことを防げます。

例
--------

### 投稿レスポンスに投稿メタを表示

```php
<?php
add_action( 'rest_api_init', 'slug_register_starship' );
function slug_register_starship() {
    register_rest_field( 'post',
        'starship',
        array(
            'get_callback'    => 'slug_get_starship',
            'update_callback' => null,
            'schema'          => null,
        )
    );
}

/**
 * "starship"フィールドの値を取得
 *
 * @param array $object 現在の投稿の詳細データ
 * @param string $field_name フィールド名
 * @param WP_REST_Request $request 現在のリクエスト
 *
 * @return mixed
 */
function slug_get_starship( $object, $field_name, $request ) {
    return get_post_meta( $object[ 'id' ], $field_name, true );
}
```

この例では投稿のレスポンスに"starship"投稿メタフィールドを追加しています。コードを簡略化するためにフィールド名が投稿メタフィールド名と対応していることに注意してください。これは必須ではありません。

### 投稿レスポンスで投稿メタフィールを読み書きする

```php
<?php
/**
 * "spaceship"フィールドを投稿のRST APIレスポンスに追加して読み書き可能にする
 */
add_action( 'rest_api_init', 'slug_register_spaceship' );
function slug_register_spaceship() {
    register_rest_field( 'post',
        'starship',
        array(
            'get_callback'    => 'slug_get_spaceship',
            'update_callback' => 'slug_update_spaceship',
            'schema'          => null,
        )
    );
}
/**
 * カスタムフィールドのデータ取得するためのハンドラ
 *
 * @since 0.1.0
 *
 * @param array $object The レスポンス・オブジェクト
 * @param string $field_name フィールド名
 * @param WP_REST_Request $request 現在のリクエスト
 *
 * @return mixed
 */
function slug_get_spaceship( $object, $field_name, $request ) {
    return get_post_meta( $object[ 'id' ], $field_name );
}

/**
 * Handler for updating custom field data.
 *
 * @since 0.1.0
 *
 * @param mixed $value フィールド名The value of the field
 * @param object $object レスポンス・オブジェクト
 * @param string $field_name フィールド名
 *
 * @return bool|int
 */
function slug_update_spaceship( $value, $object, $field_name ) {
    if ( ! $value || ! is_string( $value ) ) {
        return;
    }

    return update_post_meta( $object->ID, $field_name, strip_tags( $value ) );

}
```

この例ではどのように投稿メタフィールドを読み書きできるようにするかを示しています。これは`wp-json/wp/v2/posts/<post-id>`へのPOSTリクエストでspaceshipフィールドを更新できるようにし、`wp-json/wp/v2/posts/`へのPOSTリクエストで追加できるようにします。

### 任意の関数を返す

```php
<?php
/**
 * フィールドを追加するために任意の関数を使う
 */
add_action( 'rest_api_init', 'slug_register_something_random' );
function slug_register_something_random() {
    register_rest_field( 'post',
        'something',
        array(
            'get_callback'    => 'slug_get_something',
            'update_callback' => 'slug_update_something',
            'schema'          => null,
        )
    );
}
```

一つ前の例では、ヘルパー関数はAPIによって渡される引数を`get_post_meta`や`update_post_meta`に合わせたものでした。この例では任意の関数が呼び出されており、同じようなやり方で引数を受け取ることでしょう。

