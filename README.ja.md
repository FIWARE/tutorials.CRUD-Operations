![FIWARE Banner](https://fiware.github.io/tutorials.CRUD-Operations/img/fiware.png)

[![NGSI v2](https://img.shields.io/badge/NGSI-v2-blue.svg)](http://fiware.github.io/context.Orion/api/v2/stable/)

このチュートリアルでは、FIWARE ユーザに CRUD オペレーションについて説明します。

このチュートリアルでは、以前の[在庫管理の例](https://github.com/Fiware/tutorials.Entity-Relationships/)で作成されたデータを基に、[CRUD オペレーション](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete)の概念を紹介し、ユーザがコンテキスト内に保持されているデータをオペレーションできるようにします。

このチュートリアルでは、全体で [cUrl](https://ec.haxx.se/) コマンドを使用していますが、[Postman documentation](http://fiware.github.io/tutorials.Getting-Started/) も利用できます。

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/825950653bc2350307c3)

# 内容

- [データ・エンティティ](#data-entities)
  * [在庫管理システム内のエンティティ](#entities-within-a-stock-management-system)
- [アーキテクチャ](#architecture)
- [前提条件](#prerequisites)
  * [Docker](#docker)
  * [Cygwin](#cygwin)
- [起動](#start-up)
- [CRUD とは？](#what-is-crud)
  * [エンティティ CRUD オペレーション](#entity-crud-operations)
  * [属性 CRUD オペレーション](#attribute-crud-operations)
  * [バッチ CRUD オペレーション](#batch-crud-operations)
- [FIWARE を使用した CRUD オペレーションの例](#example-crud-operations-using-fiware)
  * [作成 (Create) オペレーション](#create-operations)
    + [新しいデータ・エンティティの作成](#create-a-new-data-entity)
    + [新しい属性の作成](#create-a-new-attribute)
    + [新しいデータ・エンティティまたは属性のバッチ作成](#batch-create-new-data-entities-or-attributes)
    + [新しいデータ・エンティティのバッチ作成/上書き](#batch-createoverwrite-new-data-entities)
  * [読み取り (Read) オペレーション](#read-operations)
    + [フィルタリング](#filtering)
    + [データエンティティ(詳細)を読み取り](#read-a-data-entity)
    + [データ・エンティティから属性の読み取り](#read-an-attribute-from-a-data-entity)
    + [データ・エンティティ(キー値のペア)の読み取り](#read-a-data-entity-key-value-pairs)
    + [データ・エンティティから複数の属性値の読み取り](#read-multiple-attributes-values-from-a-data-entity)
    + [すべてのデータ・エンティティ(詳細)を一覧表示](#list-all-data-entities-verbose)
    + [すべてのデータ・エンティティ(キー値のペア)を一覧表示](#list-all-data-entities-key-value-pairs)
    + [id でデータ・エンティティを一覧表示](#list-data-entity-by-id)
  * [更新 (Update) オペレーション](#update-operations)
    + [属性の値を上書き](#overwrite-the-value-of-an-attribute-value)
    + [データ・エンティティの複数の属性を上書き](#overwrite-multiple-attributes-of-a-data-entity)
    + [複数のデータ・エンティティの属性のバッチ上書き](#batch-overwrite-attributes-of-multiple-data-entities)
    + [複数のデータ・エンティティの属性のバッチ作成/上書き](#batch-createoverwrite-attributes-of-multiple-data-entities)
    + [エンティティ・データの一括置換](#batch-replace-entity-data)
  * [削除 (DELETE) オペレーション](#delete-operations)
    + [データの関係](#data-relationships)
    + [データ・エンティティの削除](#delete-a-data-entity)
    + [データ・エンティティからの属性の削除](#delete-an-attribute-from-a-data-entity)
    + [複数のデータ・エンティティの一括削除](#batch-delete-multiple-data-entities)
    + [データ・エンティティからの複数の属性のバッチ削除](#batch-delete-multiple-attributes-from-a-data-entity)
    + [既存のデータ関係を見つける](#find-existing-data-relationships)

<a name="data-entities"></a>
# データ・エンティティ

FIWARE プラットフォーム内では、エンティティは、実世界に存在する物理的または概念的オブジェクトの状態を表します。

<a name="entities-within-a-stock-management-system"></a>
## 在庫管理システム内のエンティティ

シンプルな在庫管理システムでは、現在4つのタイプのエンティティがあります。エンティティ間の関係は、次のように定義されます :

![](https://fiware.github.io/tutorials.Entity-Relationships/img/entities.png)

* **Store** : ストアは実世界のレンガとモルタルの建物です。ストアには次のようなプロパティがあります :
    + name : ストアの名前。例えば、"Checkpoint Markt"
    + address : ストアの住所。例えば、"Friedrichstraße 44, 10969 Kreuzberg, Berlin"
    + location : ストアの物理的なローケーション。例えば、*52.5075 N, 13.3903 E*
* **Shelf** : 棚は販売したいオブジェクトを保持するための、現実世界のデバイスです。各棚には次のようなプロパティがあります :
    + name : 棚の名前。例えば、"Wall Unit"
    + location : 棚の物理的なローケーション。例えば、*52.5075 N, 13.3903 E*
    + maximum_capacity : 棚の最大容量
    + 棚(shelf)が存在するストア(store)への関連付け
* **Product** : 製品は販売するものとして定義されています。それは概念的なオブジェクトです。製品には次のような特性があります :
    + name : 製品の名前。例えば、"Vodka"
    + price : 製品の価格。例えば、13.99ユーロ
    + size : 製品のサイズ。例えば、小さい
* **Inventory Item** : インベントリ項目は製品、店舗、棚、および物理的な物を関連付けるために使用される別の概念エンティティです。以下のようなプロパティを持ちます :
    + product : 販売されている製品へのアソシエーション
    + store : 製品が販売されている店舗への関連付け
    + shelf : 製品が展示されている棚への関連
    + stock_count : 倉庫で利用可能な製品の在庫数
    + shelf_count : 棚で利用可能な製品の在庫数


ご覧のとおり、上記で定義されたエンティティのそれぞれは、変更される可能性のあるいくつかのプロパティを含んでいます。製品の価格は変わる可能性があり、在庫が売却され、在庫の在庫数が減る可能性があります。


<a name="architecture"></a>
# アーキテクチャ

このアプリケーションは、[Orion Context Broker](https://catalogue.fiware.org/enablers/publishsubscribe-context-broker-orion-context-broker) という1つの FIWARE コンポーネントのみを使用します。アプリケーションが *"Powered by FIWARE"* と認定するには、Orion Context Broker を使用するだけで十分です。

現在、Orion Context Broker はオープンソースの [MongoDB](https://www.mongodb.com/) 技術を利用して、コンテキスト・データの永続性を維持しています。したがって、アーキテクチャは2つの要素で構成されます。

* [NGSI](http://fiware.github.io/specifications/ngsiv2/latest/) を使用してリクエストを受信するOrion Context Broker サーバ
* Orion Context Broker サーバに関連付けられている MongoDB データベース

2つの要素間のすべての対話は HTTP リクエストによって開始されるため、エンティティはコンテナ化され、公開されたポートから実行されます。


![](https://fiware.github.io/tutorials.Entity-Relationships/img/architecture.png)

<a name="prerequisites"></a>
# 前提条件

<a name="docker"></a>
## Docker

物事を単純にするために、両方のコンポーネントが [Docker](https://www.docker.com) を使用して実行されます。**Docker** は、さまざまコンポーネントをそれぞれの環境に分離することを可能にするコンテナ・テクノロジです。

* Docker を Windows にインストールするには、[こちら](https://docs.docker.com/docker-for-windows/)の手順に従ってください
* Docker を Mac にインストールするには、[こちら](https://docs.docker.com/docker-for-mac/)の手順に従ってください
* Docker を Linux にインストールするには、[こちら](https://docs.docker.com/install/)の手順に従ってください

**Docker Compose** は、マルチコンテナ Docker アプリケーションを定義して実行するためのツールです。[YAML file](htt
ps://raw.githubusercontent.com/Fiware/tutorials.Getting-Started/master/docker-compose.yml) ファイルは、アプリケーションのために必要なサービスを設定する使用されています。つまり、すべてのコンテナ・サービスは1つのコマンドで呼び出すことができます。Docker Compose は、デフォルトで Docker for Windows とD ocker for Mac の一部としてインストールされますが、Linux ユーザは[ここ](https://docs.docker.com/compose/install/)に記載されている手順に従う必要があります。


<a name="cygwin"></a>
## Cygwin 

シンプルな bash スクリプトを使用してサービスを開始します。Windows ユーザは [cygwin](http://www.cygwin.com/) をダウンロードして、Windows 上の Linux ディストリビューションと同様のコマンドライン機能を提供する必要があります。


<a name="start-up"></a>
# 起動

リポジトリ内で提供される bash スクリプトを実行すると、コマンドラインからすべてのサービスを初期化できます :

```console
./services start
```

このコマンドは、起動時に以前の[ストア・ファインダのチュートリアル](https://github.com/Fiware/tutorials.Entity-Relationships)のシードデータもインポートします。


<a name="what-is-crud"></a>
# CRUD とは？

**Create**, **Read**, **Update**, **Delete** は、永続ストレージの4つの基本機能です。これらのオペレーションは、通常、頭文字の CRUD を使用して参照されます。データベース内では、これらのオペレーションはそれぞれ一連のコマンドに直接マッピングされますが、RESTful API との関係はやや複雑です。

Orion Context Broker サーバは、[NGSI](https://fiware.github.io/specifications/OpenAPI/ngsiv2) を使用してコンテキスト・データをオペレーションします。RESTful APIとして、コンテキスト内に保持されているデータをオペレーションするリクエストは、HTTP 動詞を CRUD オペレーションにマッピングする際に見られる標準的な規則に従います。

<a name="entity-crud-operations"></a>
## エンティティ CRUD オペレーション


`<entity-id>` がコンテキスト内で認識されていない、または未指定のオペレーションでは、`/v2/entities` エンドポイントが使用されます。

`<entity-id>` がコンテキスト内で認識されると、個々のデータ・エンティティは、`/v2/entities/<entity-id>` エンドポイントを使用してオペレーションできます。

エンティティ識別子は、NGSI-LD ガイドラインに従った URN であることが推奨されます。したがって、各 `id` は、標準形式に従った URN です : `urn:ngsi-ld:<entity-type>:<entity-id>`。これは、コンテキスト・データ内のすべての `id` が一意であることを意味します。


| HTTP 動詞   | `/v2/entities`  | `/v2/entities/<entity-id>`  |
|-----------  |:--------------: |:-----------------------: |
| **POST**    | 新しいエンティティを作成し、コンテキストに追加します。 | 指定されたエンティティの属性を作成または更新します。 |
| **GET**     | コンテキストからエンティティ・データを読み込みます。これにより、複数のエンティティからのデータが返されます。データはフィルタリングできます。 | 指定されたエンティティからエンティティ・データを読み込みます。これは、単一のエンティティからのデータのみを返します。データはフィルタリングできます。 | 
| **PUT**     | :x:   | :x:   |
| **PATCH**   | :x:   | :x:   |
| **DELETE**  | :x:  | コンテキストからエンティティを削除します。 | 

エンティティ・エンドポイントの完全なリストは、[NGSI v2 Swagger Specification](https://fiware.github.io/specifications/OpenAPI/ngsiv2#/Entities) で、見つけることができます。

<a name="attribute-crud-operations"></a>
## 属性 CRUD オペレーション

属性に対して CRUD オペレーションを実行するには、`<entity-id>` を知る必要があります。各属性は実質的にキーと値 のペアです。

3つのエンドポイントがあります :

* `/v2/entities/<entity-id>/attrs` は、1つ以上の既存の属性を更新するパッチオペレーションにのみ使用されます。
* `/v2/entities/<entity-id>/attrs/<attribute>` は、属性を全体としてオペレーションするために使用されます。
* `/v2/entities/<entity-id>/attrs/<attribute>/value` は、属性の `value` を読み取りまたは更新に使用され、`type` をそのまま残します。


| HTTP 動詞   | `.../attrs`  | `.../attrs/<attribute>`  | `.../attrs/<attribute>/value`  |
|-----------  |:-----------: |:-----------------------: |:-----------------------------: |
| **POST**    |  :x:   | :x:   | :x:   |
| **GET**     |  :x:   | :x:   | 指定したエンティティから属性の値を読み込みます。これは単一のフィールドを返します。 |
| **PUT**     |  :x:   | :x:   | 指定されたエンティティから単一属性の値を更新します。 |
| **PATCH**   |  既存のエンティティから1つまたは複数の既存の属性を更新します。 | :x:   | :x:   |
| **DELETE**. |  :x: |  既存のエンティティから既存の属性を削除します。 | :x:  |

属性エンドポイントの完全なリストは、[NGSI v2 Swagger Specification](https://fiware.github.io/specifications/OpenAPI/ngsiv2#/Attributes) で見つけることができます。

<a name="batch-crud-operations"></a>
## バッチ CRUD オペレーション

さらに、Orion Context Broker は、単一オペレーションで複数のエンティティをオペレーションするためのコンビニエンス・バッチ・オペレーションのエンドポイント `/v2/op/update` を持っています。

バッチ・オペレーションは常に POST リクエストを使用して行われます。ここで、ペイロードは2つのプロパティを持つオブジェクトです :

* `actionType` : アクションの種類を呼び出すために指定します。例、`DELETE`
* `entities` :  更新するエンティティのリストを保持している配列オブジェクトです。オペレーションに使用される関連エンティティ・データと一緒に使用されます



<a name="example-crud-operations-using-fiware"></a>
# FIWARE を使用した CRUD オペレーションの例

次の例では、Orion Context Brokerが、`localhost` のポート 1026 をリッスンしていると仮定し、最初のシード・データを前のチュートリアルからインポートしました。

すべての例は、在庫管理システムで定義された **Product** エンティティを参照しています。したがって、CRUD オペレーションは、製品または一連の製品の追加、読み取り、修正および削除に関連します。これは、ストアの地域マネージャーの典型的な使用例です。たとえば、価格の設定や販売可能な製品の決定です。それぞれのケースで受け取った実際のレスポンスは、その時のシステムのコンテキスト・データの状態によって異なります。すでに間違ってエンティティを削除した場合は、コマンドラインからデータを再ロードして初期コンテキストを復元することができます。

```console
./import-data
```


<a name="create-operations"></a>
## 作成 (Create) オペレーション

作成 (Create) オペレーションは、HTTP POST へマップします。

* `/v2/entities` エンドポイントは、新しいエンティティを作成するために使用されます
* `/v2/entities/<entity>` エンドポイントは、新しい属性を追加するために使用されます

新しく作成されたエンティティは、`id` と `type` 属性を持たなければならず、それぞれの追加属性はオプションであり、記述されているシステムに依存します。それぞれの追加属性も、定義された` type` と `value` 属性をもつべきです。

オペレーションが成功した場合、レスポンスは、**204 - No Context** になるか、オペレーションが失敗した場合、**422 - Unprocessable Entity error response** になります


<a name="create-a-new-data-entity"></a>
### 新しいデータ・エンティティの作成

この例では、新しい **Product** エンティティ (99セント の "Lemonade") をコンテキストに追加します。

#### Request:

```console
curl --request POST \
  --url 'http://localhost:1026/v2/entities/' \
  --header 'Content-Type: application/json' \
  --data ' {
      "id":"urn:ngsi-ld:Product:010", "type":"Product",
      "name":{"type":"Text", "value":"Lemonade"},
      "size":{"type":"Text", "value": "S"},
      "price":{"type":"Integer", "value": 99}
}'
```

新しいエンティティは、`/v2/entities/` エンドポイントに POST リクエストを行うことで追加できます。

いずれかの属性がすでにコンテキストに存在する場合、リクエストは失敗します。

GET リクエストを行うことで、新しい製品がコンテキスト内に見つかるかどうかを確認することができます :

```console
curl --request GET \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:010'
```



<a name="create-a-new-attribute"></a>
### 新しい属性の作成

この例では、`id=urn:ngsi-ld:Product:001` の既存 Product エンティティに新しい `specialOffer` 属性を追加します。

#### Request:

```console
curl --request POST \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:001/attrs' \
  --header 'Content-Type: application/json' \
  --data '{
      "specialOffer":{"value": true}
}'
```

新しい属性は、`/v2/entities/<entity>/attrs` エンドポイントに POST リクエストを行うことで追加できます。

ペイロードは、示されているように属性名と値を保持する JSON オブジェクトで構成する必要があります。

`type` が指定されていない場合、デフォルト・タイプ(`Boolean`, `Text`, `Number` または `StructuredValue`)が割り当てられます。

同じ `id` を使用する後続のリクエストは、コンテキスト内の属性の値を更新します。

GET リクエストを行うことで、新しい **Product** 属性がコンテキスト内にあるかどうかを確認することができます :

```console
curl --request GET \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:001'
```

ご覧のように、 "Beer" **Product** エンティティにブール値の `specialOffer` フラグが付けられています。



<a name="batch-create-new-data-entities-or-attributes"></a>
### 新しいデータ・エンティティまたは属性のバッチ作成

この例では、コンビニエンス・バッチ処理エンドポイントを使用して、2つの新しい **Product** エンティティと1つの新しい属性 (`offerPrice`)をコンテキストに追加します。

#### Request:

```console
curl --request POST \
  --url 'http://localhost:1026/v2/op/update/' \
  --header 'Content-Type: application/json' \
  --data '{
  "actionType":"APPEND_STRICT",
  "entities":[
    {
      "id":"urn:ngsi-ld:Product:011", "type":"Product",
      "name":{"type":"Text", "value":"Brandy"},
      "size":{"type":"Text", "value": "M"},
      "price":{"type":"Integer", "value": 1199}
    },
    {
      "id":"urn:ngsi-ld:Product:012", "type":"Product",
      "name":{"type":"Text", "value":"Port"},
      "size":{"type":"Text", "value": "M"},
      "price":{"type":"Integer", "value": 1099}
    },
    {
      "id":"urn:ngsi-ld:Product:001", "type":"Product",
      "offerPrice":{"type":"Integer", "value": 89}
    }
  ]
}'
```

いずれかの属性がすでにコンテキストに存在する場合、リクエストは失敗します。

バッチ処理は、2つの属性のペイロードで `/v2/op/update` エンドポイントを使用します

* `actionType=APPEND_STRICT` は、すべてのエンティティ/属性が新しいリクエストにのみ成功することを意味します
* `entities` 属性は、作成したいエンティティの配列を保持しています。

`actionType=APPEND_STRICT` バッチ・オペレーションで同じデータを使用した後続のリクエストでは、エラー・レスポンスが発生します。


<a name="batch-createoverwrite-new-data-entities"></a>
### 新しいデータ・エンティティのバッチ作成/上書き

この例では、コンビニエンス・バッチ処理エンドポイントを使用して、2つの **Product** エンティティと1つの属性(`offerPrice`)をコンテキストに追加または修正します。

* エンティティがすでに存在する場合、リクエストはエンティティの属性を更新します
* エンティティが存在しない場合は、新しいエンティティが作成されます


#### Request:

```console
curl --request POST \
  --url 'http://localhost:1026/v2/op/update/' \
  --header 'Content-Type: application/json' \
  --data '{
  "actionType":"APPEND",
  "entities":[
    {
      "id":"urn:ngsi-ld:Product:011", "type":"Product",
      "name":{"type":"Text", "value":"Brandy"},
      "size":{"type":"Text", "value": "M"},
      "price":{"type":"Integer", "value": 1199}
    },
    {
      "id":"urn:ngsi-ld:Product:012", "type":"Product",
      "name":{"type":"Text", "value":"Port"},
      "size":{"type":"Text", "value": "M"},
      "price":{"type":"Integer", "value": 1099}
    }
  ]
}'
```

バッチ処理では、次の2つの属性を持つペイロードで、`/v2/op/update` エンドポイントを使用します :

* `actionType=APPEND` は、既存のエンティティが存在する場合は上書きすることを意味します
* エンティティ属性は、作成/上書きするエンティティの配列を保持します

`actionType=APPEND` バッチ処理で同じデータを使用する後続のリクエストは、最初のアプリケーションを超えて結果を変更することなく適用することができます。


<a name="read-operations"></a>
## 読み取り (Read) オペレーション

* `/v2/entities` エンドポイントは、オペレーションをリストするために使用されます
* `/v2/entities/<entity>` エンドポイントは、単一のエンティティの詳細情報を取得するために使用されています


<a name="filtering"></a>
### フィルタリング

* optionsパラメータ (attrs パラメータと組み合わせて)は、返されるフィールドをフィルタリングするために使用されます
* q パラメータは、返されるエンティティをフィルタリングするために使用できます

<a name="read-a-data-entity"></a>
### データ・エンティティ(詳細)を読み取り

この例では、既知の `id` を持つ既存 **Product** エンティティから完全なコンテキストを読み取ります。

#### Request:

```console
curl --request GET \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:010'
```

#### Response:

**Product** `urn:ngsi-ld:Product:010` は 99セントの"Lemonade" です。レスポンスは以下のようになります :

```json
{
    "id": "urn:ngsi-ld:Product:010",
    "type": "Product",
    "name": { "type": "Text","value": "Lemonade","metadata": {}},
    "price": { "type": "Integer","value": 99,"metadata": {}},
    "size": { "type": "Text","value": "S","metadata": {}}
}
```

コンテキスト・データは、`/v2/entities/<entity>` エンドポイントに GET リクエストを出すことによって取り出すことができます。


<a name="read-an-attribute-from-a-data-entity"></a>
### データ・エンティティから属性の読み取り

この例では、既知の `id` を持つ既存 **Product** エンティティから単一の属性(`name`)の値を読み取ります。

#### Request:

```console
curl --request GET \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:001/attrs/name/value'
```

#### Response:

**Product** `urn:ngsi-ld:Product:001` は 99セントの"Beer"です。レスポンスは以下のようになります :

```json
"Beer"
```

コンテキスト・データは、`/v2/entities/<entity>/attrs/<attribute>/value` エンドポイントに GET リクエストを出すことによって取り出すことができます。


<a name="read-a-data-entity-key-value-pairs"></a>
### データ・エンティティ(キー値のペア)の読み取り

この例では、リクエストされた2つの属性(`name` および `price` )のキーと値のペアを、既知の `id` を持つ既存 **Product** エンティティのコンテキストから読み取ります。

#### Request:

```console
curl --request GET \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:001/?options=keyValues&attrs=name,price'
```

#### Response:

**Product** `urn:ngsi-ld:Product:001` は 99セントの"Beer"です。レスポンスは以下のようになります :

```json
{
    "id": "urn:ngsi-ld:Product:001",
    "type": "Product",
    "name": "Beer",
    "price": 99
}
```

`options=keyValues` パラメータと `attrs` パラメータを組み合わることで、キーと値のペアを取得します。



<a name="read-multiple-attributes-values-from-a-data-entity"></a>
### データ・エンティティから複数の属性値の読み取り

この例では、既知の `id` を持つ既存 **Product** エンティティのコンテキストから、2つのリクエストされた属性(`name` および `price`)の値を読み取ります。

#### Request:

```console
curl --request GET \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:001/?options=values&attrs=name,price'
```

#### Response:

**Product** `urn:ngsi-ld:Product:001` は 99セントの"Beer"です。レスポンスは以下のようになります :

```json
[
    "Beer",
    99
]
```

`options=values` パラメータと `attrs` パラメータを組み合わせることで、配列内の値のリストを返します。



<a name="list-all-data-entities-verbose"></a>
### すべてのデータ・エンティティ(詳細)を一覧表示

この例では、すべての **Product** エンティティの完全なコンテキストをリストしています。

#### Request:

```console
curl --request GET \
  --url 'http://localhost:1026/v2/entities/?type=Product'
```

### Response:

起動すると、コンテキストは9つの製品を含み、3つの製品が作成オペレーションによって追加され、フル・コンテキストでは12の製品が返されます。

```json
[
    {
        "id": "urn:ngsi-ld:Product:001",
        "type": "Product",
        "name": {"type": "Text","value": "Beer","metadata": {}},
        "offerPrice": {"type": "Integer","value": 89,"metadata": {}},
        "price": {"type": "Integer","value": 99,"metadata": {}},
        "size": {"type": "Text","value": "S","metadata": {}},
        "specialOffer": {"type": "Boolean","value": true,"metadata": {}}
    },
    {
        "id": "urn:ngsi-ld:Product:002",
        "type": "Product",
        "name": {"type": "Text","value": "Red Wine","metadata": {}},
        "price": {"type": "Integer","value": 1099,"metadata": {}},
        "size": {"type": "Text","value": "M","metadata": {}}
    },
    {
        "id": "urn:ngsi-ld:Product:003",
        "type": "Product",
        "name": {"type": "Text","value": "White Wine","metadata": {}},
        "price": {"type": "Integer","value": 1499,"metadata": {}},
        "size": {"type": "Text","value": "M","metadata": {}}
    },
    {
        "id": "urn:ngsi-ld:Product:004",
        "type": "Product",
        "name": {"type": "Text","value": "Vodka","metadata": {}},
        "price": {"type": "Integer","value": 5000,"metadata": {}},
        "size": {"type": "Text","value": "XL","metadata": {}}
    },
    {
        "id": "urn:ngsi-ld:Product:005",
        "type": "Product",
        "name": {"type": "Text","value": "Lager","metadata": {}},
        "price": {"type": "Integer","value": 99,"metadata": {}},
        "size": {"type": "Text","value": "S","metadata": {}}
    },
    {
        "id": "urn:ngsi-ld:Product:006",
        "type": "Product",
        "name": {"type": "Text","value": "Whisky","metadata": {}},
        "price": {"type": "Integer","value": 99,"metadata": {}},
        "size": {"type": "Text","value": "S","metadata": {}}
    },
    {
        "id": "urn:ngsi-ld:Product:007",
        "type": "Product",
        "name": {"type": "Text","value": "Gin","metadata": {}},
        "price": {"type": "Integer","value": 99,"metadata": {}},
        "size": {"type": "Text","value": "S","metadata": {}}
    },
    {
        "id": "urn:ngsi-ld:Product:008",
        "type": "Product",
        "name": {"type": "Text","value": "Apple Juice","metadata": {}},
        "price": {"type": "Integer","value": 99,"metadata": {}},
        "size": {"type": "Text","value": "S","metadata": {}}
    },
    {
        "id": "urn:ngsi-ld:Product:009",
        "type": "Product",
        "name": {"type": "Text","value": "Orange Juice","metadata": {}},
        "price": {"type": "Integer","value": 99,"metadata": {}},
        "size": {"type": "Text","value": "S","metadata": {}}
    },
    {
        "id": "urn:ngsi-ld:Product:010",
        "type": "Product",
        "name": {"type": "Text","value": "Lemonade","metadata": {}},
        "price": {"type": "Integer","value": 99,"metadata": {}},
        "size": {"type": "Text","value": "S","metadata": {}}
    },
    {
        "id": "urn:ngsi-ld:Product:011",
        "type": "Product",
        "name": {"type": "Text","value": "Brandy","metadata": {}},
        "price": {"type": "Integer","value": 1199,"metadata": {}},
        "size": {"type": "Text","value": "M","metadata": {}}
    },
    {
        "id": "urn:ngsi-ld:Product:012",
        "type": "Product",
        "name": {"type": "Text","value": "Port","metadata": {}},
        "price": {"type": "Integer","value": 1099,"metadata": {}},
        "size": {"type": "Text","value": "M","metadata": {}}
    }
]
```

<a name="list-all-data-entities-key-value-pairs"></a>
### すべてのデータ・エンティティ(キー値のペア)を一覧表示

この例では、すべての **Product** エンティティの `name` と `price` 属性を示します。

#### Request:

```console
curl --request GET \
  --url 'http://localhost:1026/v2/entities/?type=Product&options=keyValues&attrs=name,price'
```

#### Response:
起動すると、コンテキストは9つの製品を含み、3つの製品が作成オペレーションによって追加され、フル・コンテキストでは12の製品が返されます。

```json
[
    {
        "id": "urn:ngsi-ld:Product:001",
        "type": "Product",
        "name": "Beer",
        "price": 99
    },
    {
        "id": "urn:ngsi-ld:Product:002",
        "type": "Product",
        "name": "Red Wine",
        "price": 1099
    },
    {
        "id": "urn:ngsi-ld:Product:003",
        "type": "Product",
        "name": "White Wine",
        "price": 1499
    },
    {
        "id": "urn:ngsi-ld:Product:004",
        "type": "Product",
        "name": "Vodka",
        "price": 5000
    },
    {
        "id": "urn:ngsi-ld:Product:005",
        "type": "Product",
        "name": "Lager",
        "price": 99
    },
    {
        "id": "urn:ngsi-ld:Product:006",
        "type": "Product",
        "name": "Whisky",
        "price": 99
    },
    {
        "id": "urn:ngsi-ld:Product:007",
        "type": "Product",
        "name": "Gin",
        "price": 99
    },
    {
        "id": "urn:ngsi-ld:Product:008",
        "type": "Product",
        "name": "Apple Juice",
        "price": 99
    },
    {
        "id": "urn:ngsi-ld:Product:009",
        "type": "Product",
        "name": "Orange Juice",
        "price": 99
    },
    {
        "id": "urn:ngsi-ld:Product:010",
        "type": "Product",
        "name": "Lemonade",
        "price": 99
    },
    {
        "id": "urn:ngsi-ld:Product:011",
        "type": "Product",
        "name": "Brandy",
        "price": 1199
    },
    {
        "id": "urn:ngsi-ld:Product:012",
        "type": "Product",
        "name": "Port",
        "price": 1099
    }
]
```

指定されたエンティティ・タイプのフル・コンテキスト・データは、`/v2/entities/` エンドポイントへの GET リクエストを行い、`type` パラメータを指定することによって取得できます。`options=keyValues` パラメータおよび `attrs` パラメータと組み合わせて、キーと値を取得します。


<a name="list-data-entity-by-id"></a>
### id でデータ・エンティティを一覧表示

この例では、すべての **Prodcut** エンティティの `id` と `type` が一覧表示されます。

#### Request:

```console
curl --request GET \
  --url 'http://localhost:1026/v2/entities/?type=Product&options=count&attrs=id'
```

#### Response:
起動すると、コンテキストは9つの製品を含み、3つの製品が作成オペレーションによって追加され、フル・コンテキストでは12の製品が返されます。

```json
[
    {
        "id": "urn:ngsi-ld:Product:001",
        "type": "Product"
    },
    {
        "id": "urn:ngsi-ld:Product:002",
        "type": "Product"
    },
    {
        "id": "urn:ngsi-ld:Product:003",
        "type": "Product"
    },
    {
        "id": "urn:ngsi-ld:Product:004",
        "type": "Product"
    },
    {
        "id": "urn:ngsi-ld:Product:005",
        "type": "Product"
    },
    {
        "id": "urn:ngsi-ld:Product:006",
        "type": "Product"
    },
    {
        "id": "urn:ngsi-ld:Product:007",
        "type": "Product"
    },
    {
        "id": "urn:ngsi-ld:Product:008",
        "type": "Product"
    },
    {
        "id": "urn:ngsi-ld:Product:009",
        "type": "Product"
    },
    {
        "id": "urn:ngsi-ld:Product:010",
        "type": "Product"
    },
    {
        "id": "urn:ngsi-ld:Product:011",
        "type": "Product"
    },
    {
        "id": "urn:ngsi-ld:Product:012",
        "type": "Product"
    }
]
```

指定されたエンティティ・タイプのコンテキスト・データは、`/v2/entities/` エンドポイントへの GET リクエストを行い、`type` パラメータを指定することによって取得できます。これと、`options=count`, `attrs=id` を組み合わせidて指定された `type` の `id` 属性を返します。


<a name="update-operations"></a>
## 更新 (Update) オペレーション

上書きオペレーションは HTTP PUT にマップされます。HTTP PATCH を使用すると、一度に複数の属性を更新できます。

* `/v2/entities/<entity>/attrs/<attribute>/value` エンドポイントは属性を更新するために使用されます
* `/v2/entities/<entity>/attrs` エンドポイントは、複数の属性を更新するために使用されます


<a name="overwrite-the-value-of-an-attribute-value"></a>
### 属性の値を上書き

この例では、`id=urn:ngsi-ld:Product:001` を持つ Entity の `price` 属性の値を次のように更新します。 

#### Request:

```console
curl --request PUT \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:001/attrs/price/value' \
  --header 'Content-Type: text/plain' \
  --data 89
```

既存の属性値は、`/v2/entities/<entity>/attrs/<attribute>/value` エンドポイントに PUT リクエストを行うことで変更できます。


<a name="overwrite-multiple-attributes-of-a-data-entity"></a>
### データ・エンティティの複数の属性を上書き

この例は、`id=urn:ngsi-ld:Product:001` を持つ、`Entity` の `price` 属性と `name` 属性の両方の値を同時に更新します。 

#### Request:

```console
curl --request PATCH \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:001/attrs' \
  --header 'Content-Type: application/json' \
  --data ' {
      "price":{"type":"Integer", "value": 89}
}'
```


<a name="batch-overwrite-attributes-of-multiple-data-entities"></a>
### 複数のデータ・エンティティの属性のバッチ上書き

この例では、コンビニエンス・バッチ処理エンドポイントを使用して一連の使用可能な製品を作成しています。

#### Request:

```console
curl --request POST \
  --url 'http://localhost:1026/v2/op/update' \
  --header 'Content-Type: application/json' \
  --data '{
  "actionType":"UPDATE",
  "entities":[
    {
      "id":"urn:ngsi-ld:Product:001", "type":"Product",
      "price":{"type":"Integer", "value": 1199}
    },
    {
      "id":"urn:ngsi-ld:Product:002", "type":"Product",
      "price":{"type":"Integer", "value": 1199},
      "size": {"type":"Text", "value": "L"}
    }
  ]
}'
```

バッチ処理では、2つの属性のペイロードを持つ `/v2/op/update` エンドポイントを使用します。

- `actionType = APPEND` は、既存のエンティティが存在する場合はそれを上書きすることを意味しますが、`entities` 属性は更新したいエンティティの配列を保持します。


<a name="batch-createoverwrite-attributes-of-multiple-data-entities"></a>
### 複数のデータ・エンティティの属性のバッチ作成/上書き

この例では、コンビニエンス・バッチ処理エンドポイントを使用して一連の利用可能な製品を作成しています。

#### Request:

```console
curl --request POST \
  --url 'http://localhost:1026/v2/op/update' \
  --header 'Content-Type: application/json' \
  --data '{
  "actionType":"APPEND",
  "entities":[
    {
      "id":"urn:ngsi-ld:Product:001", "type":"Product",
      "price":{"type":"Integer", "value": 1199}
    },
    {
      "id":"urn:ngsi-ld:Product:002", "type":"Product",
      "price":{"type":"Integer", "value": 1199},
      "specialOffer": {"type":"Boolean", "value":  true}
    }
  ]
}'
```

バッチ処理では、2つの属性のペイロードを持つ `/v2/op/update` エンドポイントを使用します。

- `actionType = APPEND` は、既存のエンティティが存在する場合はそれを上書きすることを意味しますが、`entities` 属性は更新したいエンティティの配列を保持します。



<a name="batch-replace-entity-data"></a>
### エンティティ・データの一括置換

この例では、コンビニエンス・バッチ処理エンドポイントを使用して一連の使用可能な製品を作成しています。

#### Request:

```console
curl --request POST \
  --url 'http://localhost:1026/v2/op/update' \
  --header 'Content-Type: application/json' \
  --data '{
  "actionType":"REPLACE",
  "entities":[
    {
      "id":"urn:ngsi-ld:Product:010", "type":"Product",
      "price":{"type":"Integer", "value": 1199}
    }
  ]
}'
```

バッチ処理では、2つの属性のペイロードを持つ `/v2/op/update` エンドポイントを使用します。

- `actionType=REPLACE` は、既存のエンティティが存在する場合は上書きすることを意味しますが、`entities` 属性は更新するエンティティの配列を保持します。


<a name="delete-operations"></a>
## 削除 (DELETE) オペレーション
削除 (Delete) オペレーション は、HTTP DELETE にマップします。

- `/v2/entities/<entity>` エンドポイントは、エンティティを削除するために使用されます
- `/v2/entities/<entity>/attrs/<attribute>` エンドポイントは、属性を削除するために使用されます

レスポンスは、オペレーションが成功すれば、**204 - No Content** となり、失敗すれば、**404 - Not Found** となります。


<a name="data-relationships"></a>
### データのリレーションシップ
コンテキスト内に互いに関係するエンティティがある場合、エンティティを削除するときは注意が必要です。エンティティが削除されるとリファレンスが残っていないことを確認する必要があります。

カスケード削除の構成はこのチュートリアルの範囲を超えていますが、バッチ削除リクエストを使用することもできます。

<a name="delete-a-data-entity"></a>
### データ・エンティティの削除

この例では、`id=urn:ngsi-ld:Product:001` のエンティティをコンテキストから削除します。

#### Request:

```console
curl --request DELETE \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:010'
```

エンティティは、`/v2/entities/<entity>` エンドポイントに DELETE リクエストを行うことによって削除されます。

同じ `id` を使用する後続のリクエストは、エンティティがもはやコンテキストに存在しないため、エラーレスポンスになります。


<a name="delete-an-attribute-from-a-data-entity"></a>
### データ・エンティティからの属性の削除

この例では、`id=urn:ngsi-ld:Product:010` のエンティティから `specialOffer` 属性を削除します。

#### Request:

```console
curl --request DELETE \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:010/attrs/specialOffer'
```

属性は、`/v2/entities/<entity>/attrs/<attribute>` エンドポイントに DELETE リクエストを行うことによって削除できます。

属性がコンテキストに存在しない場合は、エラーレスポンスが返されます。

<a name="batch-delete-multiple-data-entities"></a>
### 複数のデータ・エンティティの一括削除

この例では、コンビニエンス・バッチ処理エンドポイントを使用して一連の利用可能な `Product` エンティティを削除します。

#### Request:

```console
curl --request POST \
  --url 'http://localhost:1026/v2/op/update' \
  --header 'Content-Type: application/json' \
  --data '{
  "actionType":"DELETE",
  "entities":[
    {
      "id":"urn:ngsi-ld:Product:001", "type":"Product"
    },
    {
      "id":"urn:ngsi-ld:Product:002", "type":"Product"
    }
  ]
}'
```

バッチ処理では、2つの属性のペイロードを持つ `/v2/op/update` エンドポイントを使用します。

- `actionType=DELETE` は、コンテキストから何かを削除することを意味し、`entities`属性は、更新するエンティティの `id` を保持します。

コンテキスト内にエンティティが存在しない場合は、エラーレスポンスが返されます。

<a name="batch-delete-multiple-attributes-from-a-data-entity"></a>
### データ・エンティティからの複数の属性のバッチ削除

この例では、コンビニエンス・バッチ処理エンドポイントを使用して、使用可能な **Product** エンティティから一連の属性を削除します。

#### Request:

```console
curl --request POST \
  --url 'http://localhost:1026/v2/op/update' \
  --header 'Content-Type: application/json' \
  --data '{
  "actionType":"DELETE",
  "entities":[
    {
      "id":"urn:ngsi-ld:Product:010", "type":"Product",
      "price":{},
      "name": {}
    }
  ]
}'
```

バッチ処理では、2つの属性のペイロードを持つ `/v2/op/update` エンドポイントを使用します。

- `actionType=DELETE` は、コンテキストから何かを削除することを意味し、`entities`属性は、更新する属性の 配列 を保持します。

コンテキストに属性が存在しない場合、結果はエラーレスポンスになります。

<a name="find-existing-data-relationships"></a>
### 既存のデータ・リレーションシップを見つける

この例では、`urn:ngsi-ld:Product:001` に直接関連するすべてのエンティティのキーを返します。

#### Request:

```console
curl --request GET \
  --url 'http://localhost:1026/v2/entities/?q=refProduct==urn:ngsi-ld:Product:001&options=count&attrs=type'
```

#### Response:

```json
[
    {
        "id": "urn:ngsi-ld:InventoryItem:001",
        "type": "InventoryItem"
    },
    {
        "id": "urn:ngsi-ld:InventoryItem:004",
        "type": "InventoryItem"
    },
    {
        "id": "urn:ngsi-ld:InventoryItem:006",
        "type": "InventoryItem"
    },
    {
        "id": "urn:ngsi-ld:InventoryItem:401",
        "type": "InventoryItem"
    }
]
```

* このリクエストによって空の配列が返された場合、エンティティには関連付けられていません。安全に削除できます
* レスポンスが一連の `InventoryItem` エンティティをリストする場合、関連する `Product` エンティティがコンテキストから削除される前に削除する必要があります

<a name="next-steps"></a>
# 次のステップ

アドバンス機能を追加するアプリをもっと複雑にする方法を知りたいですか？ このシリーズの他のチュートリアルを読むことで、学ぶことができます。

&nbsp; 101. [Getting Started](https://github.com/Fiware/tutorials.Getting-Started)<br/>
&nbsp; 102. [Entity Relationships](https://github.com/Fiware/tutorials.Entity-Relationships/)<br/>
&nbsp; 103. [CRUD Operations](https://github.com/Fiware/tutorials.CRUD-Operations/)<br/>
&nbsp; 104. [Context Providers](https://github.com/Fiware/tutorials.Context-Providers/)<br/>
&nbsp; 105. [Altering the Context Programmatically](https://github.com/Fiware/tutorials.Accessing-Context/)<br/> 
&nbsp; 106. [Subscribing to Changes in Context](https://github.com/Fiware/tutorials.Subscriptions/)<br/>

&nbsp; 201. [Introduction to IoT Sensors](https://github.com/Fiware/tutorials.IoT-Sensors/)<br/> 
