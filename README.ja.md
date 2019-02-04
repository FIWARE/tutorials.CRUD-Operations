[![FIWARE Banner](https://fiware.github.io/tutorials.CRUD-Operations/img/fiware.png)](https://www.fiware.org/developers)

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://www.fiware.org/developers/catalogue/)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.CRUD-Operations.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://nexus.lab.fiware.org/repository/raw/public/badges/stackoverflow/fiware.svg)](https://stackoverflow.com/questions/tagged/fiware)
[![NGSI v2](https://img.shields.io/badge/NGSI-v2-blue.svg)](https://fiware-ges.github.io/core.Orion/api/v2/stable/)
<br/>
[![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)

<!-- prettier-ignore -->

このチュートリアルでは、FIWARE ユーザに CRUD オペレーションについて説明します。
チュートリアルでは、以前
の[在庫管理の例](https://github.com/Fiware/tutorials.Entity-Relationships/)で作
成されたデータを基に
、[CRUD オペレーション](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete)の
概念を紹介し、ユーザがコンテキスト内に保持されているデータをオペレーションできる
ようにします。

このチュートリアルでは、全体で [cUrl](https://ec.haxx.se/) コマンドを使用してい
ますが
、[Postman documentation](https://fiware.github.io/tutorials.Getting-Started/)
も利用できます。

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/7764c9cbc3cfe2d5b403)

## 内容

<details>
<summary>詳細 <b>(クリックして拡大)</b></summary>

-   [データ・エンティティ](#data-entities)
    -   [在庫管理システム内のエンティティ](#entities-within-a-stock-management-system)
-   [アーキテクチャ](#architecture)
-   [前提条件](#prerequisites)
    -   [Docker](#docker)
    -   [Cygwin](#cygwin)
-   [起動](#start-up)
-   [CRUD とは？](#what-is-crud)
    -   [エンティティ CRUD オペレーション](#entity-crud-operations)
    -   [属性 CRUD オペレーション](#attribute-crud-operations)
    -   [バッチ CRUD オペレーション](#batch-crud-operations)
-   [FIWARE を使用した CRUD オペレーションの例](#example-crud-operations-using-fiware)
    -   [作成 (Create) オペレーション](#create-operations)
        -   [新しいデータ・エンティティの作成](#create-a-new-data-entity)
        -   [新しい属性の作成](#create-a-new-attribute)
        -   [新しいデータ・エンティティまたは属性のバッチ作成](#batch-create-new-data-entities-or-attributes)
        -   [新しいデータ・エンティティのバッチ作成/上書き](#batch-createoverwrite-new-data-entities)
    -   [読み取り (Read) オペレーション](#read-operations)
        -   [フィルタリング](#filtering)
        -   [データエンティティ(詳細)を読み取り](#read-a-data-entity)
        -   [データ・エンティティから属性の読み取り](#read-an-attribute-from-a-data-entity)
        -   [データ・エンティティ(キー値のペア)の読み取り](#read-a-data-entity-key-value-pairs)
        -   [データ・エンティティから複数の属性値の読み取り](#read-multiple-attributes-values-from-a-data-entity)
        -   [すべてのデータ・エンティティ(詳細)を一覧表示](#list-all-data-entities-verbose)
        -   [すべてのデータ・エンティティ(キー値のペア)を一覧表示](#list-all-data-entities-key-value-pairs)
        -   [ID でデータ・エンティティを一覧表示](#list-data-entity-by-id)
    -   [更新 (Update) オペレーション](#update-operations)
        -   [属性の値を上書き](#overwrite-the-value-of-an-attribute-value)
        -   [データ・エンティティの複数の属性を上書き](#overwrite-multiple-attributes-of-a-data-entity)
        -   [複数のデータ・エンティティの属性のバッチ上書き](#batch-overwrite-attributes-of-multiple-data-entities)
        -   [複数のデータ・エンティティの属性のバッチ作成/上書き](#batch-createoverwrite-attributes-of-multiple-data-entities)
        -   [エンティティ・データの一括置換](#batch-replace-entity-data)
    -   [削除 (DELETE) オペレーション](#delete-operations)
        -   [データの関係](#data-relationships)
        -   [エンティティの削除](#delete-an-entity)
        -   [エンティティからの属性の削除](#delete-an-attribute-from-an-entity)
        -   [複数のエンティティの一括削除](#batch-delete-multiple-entities)
        -   [エンティティからの複数の属性のバッチ削除](#batch-delete-multiple-attributes-from-an-entity)
        -   [既存のデータ関係を見つける](#find-existing-data-relationships)
-   [次のステップ](#next-steps)

</details>

<a name="data-entities"></a>

# データ・エンティティ

FIWARE プラットフォーム内では、エンティティは、実世界に存在する物理的または概念
的オブジェクトの状態を表します。

<a name="entities-within-a-stock-management-system"></a>

## 在庫管理システム内のエンティティ

シンプルな在庫管理システムでは、現在 4 つのエンティティ型があります。エン
ティティ間の関係は、次のように定義されます :

![](https://fiware.github.io/tutorials.Entity-Relationships/img/entities.png)

-   **Store** : ストアは実世界のレンガとモルタルの建物です。ストアには次のような
    プロパティがあります :
    -   name : ストアの名前。例えば、"Checkpoint Markt"
    -   address : ストアの住所。例えば、"Friedrichstraße 44, 10969 Kreuzberg,
        Berlin"
    -   location : ストアの物理的なローケーション。例えば、_52.5075 N, 13.3903
        E_
-   **Shelf** : 棚は販売したいオブジェクトを保持するための、現実世界の
    オブジェクトです。各棚には次のようなプロパティがあります :
    -   name : 棚の名前。例えば、"Wall Unit"
    -   location : 棚の物理的なローケーション。例えば、_52.5075 N, 13.3903 E_
    -   maximum_capacity : 棚の最大容量
    -   棚(shelf)が置かれているストア(store)への関連付け
-   **Product** : 製品は販売するものとして定義されています。それは概念的なオブジ
    ェクトです。製品には次のような特性があります :
    -   name : 製品の名前。例えば、"Vodka"
    -   price : 製品の価格。例えば、13.99 ユーロ
    -   size : 製品のサイズ。例えば、小さい
-   **Inventory Item** : インベントリ項目は製品、店舗、棚、および物理的な物を関
    連付けるために使用される別の概念エンティティです。以下のようなプロパティを持
    ちます :
    -   product : 販売されている製品へのアソシエーション
    -   store : 製品が販売されている店舗への関連付け
    -   shelf : 製品が展示されている棚への関連
    -   stock_count : 倉庫で利用可能な製品の在庫数
    -   shelf_count : 棚で利用可能な製品の在庫数

ご覧のとおり、上記で定義されたエンティティには、変更される可能性のある
プロパティがいつか含まれています。たとえば、製品価格が変化し、在庫が売却され、
棚の商品数が減少する可能性があります。

<a name="architecture"></a>

# アーキテクチャ

このアプリケーションは
、[Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) という
1 つの FIWARE コンポーネントのみを使用します。アプリケーションが _"Powered by
FIWARE"_ と認定するには、Orion Context Broker を使用するだけで十分です。

現在、Orion Context Broker はオープンソースの

現在、Orion Context Broker はオープンソースの
[MongoDB](https://www.mongodb.com/) テクノロジに依存して、
管理しているコンテキストデータを格納しています。
したがって、アーキテクチャは 2つの要素で構成されます :

-   [NGSI](https://fiware.github.io/specifications/OpenAPI/ngsiv2) を使用してリ
    クエストを受信する
    [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/)
-   バックエンドの [MongoDB](https://www.mongodb.com/) データベース
    -   Orion Context Broker が、データ・エンティティなどのコンテキスト
        情報、サブスクリプション、登録などをストアするために使用します

2つのコンポーネントは HTTP リクエストによって相互作用するため、
コンテナ化して公開されたポートから実行できます。

![](https://fiware.github.io/tutorials.CRUD-Operations/img/architecture.png)

必要な設定情報は、関連する `docker-compose.yml` ファイルの services セクションに
あります:

```yaml
orion:
    image: fiware/orion:latest
    hostname: orion
    container_name: orion
    depends_on:
        - mongo-db
    networks:
        - default
    expose:
        - "1026"
    ports:
        - "1026:1026"
    command: -dbhost mongo-db -logLevel DEBUG
```

```yaml
mongo-db:
    image: mongo:3.6
    hostname: mongo-db
    container_name: db-mongo
    expose:
        - "27017"
    ports:
        - "27017:27017"
    networks:
        - default
    command: --bind_ip_all --smallfiles
```

どちらのコンテナも同じネットワークに常駐しています。Orion Context Broker は
ポート `1026` でリッスンしており、MongoDB はデフォルトポート `271071` で
リッスンしています。このチュートリアルでは、ネットワークの外部から2つのポートを
利用できるようにして、cUrl または Postman がネットワーク内から実行することなく
アクセスできるようにしました。コマンドラインの初期化は、一目瞭然でなければなりま
せん。

<a name="prerequisites"></a>

# 前提条件

<a name="docker"></a>

## Docker

物事を単純にするために、両方のコンポーネントが [Docker](https://www.docker.com)
を使用して実行されます。**Docker** は、環境ごとに各コンポーネントをパッケージ
化し、独立して実行することができるコンテナ・テクノロジです。

-   Docker を Windows にインストールするには
    、[こちら](https://docs.docker.com/docker-for-windows/)の手順に従ってくださ
    い
-   Docker を Mac にインストールするには
    、[こちら](https://docs.docker.com/docker-for-mac/)の手順に従ってください
-   Docker を Linux にインストールするには
    、[こちら](https://docs.docker.com/install/)の手順に従ってください

**Docker Compose** は、マルチコンテナ Docker アプリケーションを定義して実行する
ためのツールです
。[YAML file](https://raw.githubusercontent.com/Fiware/tutorials.CRUD-Operations/master/docker-compose.yml)
ファイルは、アプリケーションのために必要なサービスを構成するために使用します。つ
まり、すべてのコンテナ・サービスは 1 つのコマンドで呼び出すことができます
。Docker Compose は、デフォルトで Docker for Windows と D ocker for Mac の一部と
してインストールされますが、Linux ユーザ
は[ここ](https://docs.docker.com/compose/install/)に記載されている手順に従う必要
があります。

次のコマンドを使用して、現在の **Docker** バージョンと **Docker Compose** バージ
ョンを確認できます :

```console
docker-compose -v
docker version
```

Docker バージョン 18.03 以降と Docker Compose 1.21 以上を使用していることを確認
し、必要に応じてアップグレードしてください。

<a name="cygwin"></a>

## Cygwin

シンプルな bash スクリプトを使用してサービスを開始します。Windows ユーザは
[cygwin](http://www.cygwin.com/) をダウンロードして、Windows 上の Linux ディスト
リビューションと同様のコマンドライン機能を提供する必要があります。

<a name="start-up"></a>

# 起動

リポジトリ内で提供される bash スクリプトを実行すると、コマンドラインからすべての
サービスを初期化できます。リポジトリを複製し、以下のコマンドを実行して必要なイメ
ージを作成してください :

```console
git clone git@github.com:Fiware/tutorials.CRUD-Operations.git
cd tutorials.CRUD-Operations

./services start
```

このコマンドは、起動時に以前
の[ストア・ファインダのチュートリアル](https://github.com/Fiware/tutorials.Entity-Relationships)の
シードデータもインポートします。

<a name="what-is-crud"></a>

# CRUD とは？

**Create**, **Read**, **Update**, **Delete** は、永続ストレージの 4 つの基本機能
です。これらのオペレーションは、通常、頭文字の CRUD を使用して参照されます。デー
タベース内では、これらのオペレーションはそれぞれ一連のコマンドに直接マッピングさ
れますが、RESTful API との関係はやや複雑です。

[Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) は
、[NGSI](https://fiware.github.io/specifications/OpenAPI/ngsiv2) を使用してコン
テキスト・データをオペレーションします。RESTful API として、コンテキスト内に保持
されているデータをオペレーションするリクエストは、HTTP 動詞を CRUD オペレーショ
ンにマッピングする際に見られる標準的な規則に従います。

<a name="entity-crud-operations"></a>

## エンティティ CRUD オペレーション

`<entity-id>` がコンテキスト内で認識されていない、または未指定のオペレーションで
は、`/v2/entities` エンドポイントが使用されます。

`<entity-id>` がコンテキスト内で認識されると、個々のデータ・エンティティは
、`/v2/entities/<entity-id>` エンドポイントを使用してオペレーションできます。

エンティティ識別子は、
[NGSI-LD guidelines](https://docbox.etsi.org/ISG/CIM/Open/ISG_CIM_NGSI-LD_API_Draft_for_public_review.pdf)
ガイドラインに従った URNs であることが推奨されます。
したがって、各 `id` は、標準形式に従った URN です :
`urn:ngsi-ld:<entity-type>:<entity-id>`。これは、コンテキスト・データ内のすべて
の `id` を一意にするのに役立ちます。

| HTTP 動詞  |                                                                `/v2/entities`                                                                |                                                              `/v2/entities/<entity-id>`                                                              |
| ---------- | :------------------------------------------------------------------------------------------------------------------------------------------: | :--------------------------------------------------------------------------------------------------------------------------------------------------: |
| **POST**   |                                            新しいエンティティを作成し、コンテキストに追加します。                                            |                                                 指定されたエンティティの属性を作成または更新します。                                                 |
| **GET**    | コンテキストからエンティティ・データを読み込みます。これにより、複数のエンティティからのデータが返されます。データはフィルタリングできます。 | 指定されたエンティティからエンティティ・データを読み込みます。これは、単一のエンティティからのデータのみを返します。データはフィルタリングできます。 |
| **PUT**    |                                                                     :x:                                                                      |                                                                         :x:                                                                          |
| **PATCH**  |                                                                     :x:                                                                      |                                                                         :x:                                                                          |
| **DELETE** |                                                                     :x:                                                                      |                                                      コンテキストからエンティティを削除します。                                                      |

エンティティ・エンドポイントの完全なリストは
、[NGSI v2 Swagger Specification](https://fiware.github.io/specifications/OpenAPI/ngsiv2#/Entities)
で、見つけることができます。

<a name="attribute-crud-operations"></a>

## 属性 CRUD オペレーション

属性に対して CRUD オペレーションを実行するには、`<entity-id>` を知る必要がありま
す。各属性は実質的にキーと値 のペアです。

3 つのエンドポイントがあります :

-   `/v2/entities/<entity-id>/attrs` は、1 つ以上の既存の属性を更新するパッチオ
    ペレーションにのみ使用されます。
-   `/v2/entities/<entity-id>/attrs/<attribute>` は、属性を全体としてオペレーシ
    ョンするために使用されます。
-   `/v2/entities/<entity-id>/attrs/<attribute>/value` は、属性の `value` を読み
    取りまたは更新に使用され、`type` をそのまま残します。

| HTTP 動詞   |                           `.../attrs`                           |            `.../attrs/<attribute>`             |                           `.../attrs/<attribute>/value`                            |
| ----------- | :-------------------------------------------------------------: | :--------------------------------------------: | :--------------------------------------------------------------------------------: |
| **POST**    |                               :x:                               |                      :x:                       |                                        :x:                                         |
| **GET**     |                               :x:                               |                      :x:                       | 指定したエンティティから属性の値を読み込みます。これは単一のフィールドを返します。 |
| **PUT**     |                               :x:                               |                      :x:                       |                指定されたエンティティから単一属性の値を更新します。                |
| **PATCH**   | 既存のエンティティから 1 つまたは複数の既存の属性を更新します。 |                      :x:                       |                                        :x:                                         |
| **DELETE**. |                               :x:                               | 既存のエンティティから既存の属性を削除します。 |                                        :x:                                         |

属性エンドポイントの完全なリストは
、[NGSI v2 Swagger Specification](https://fiware.github.io/specifications/OpenAPI/ngsiv2#/Attributes)
で見つけることができます。

<a name="batch-crud-operations"></a>

## バッチ CRUD オペレーション

さらに、Orion Context Broker は、単一オペレーションで複数のエンティティをオペレ
ーションするためのコンビニエンス・バッチ・オペレーションのエンドポイント
`/v2/op/update` を持っています。


バッチ・オペレーションは、ペイロードが2つのプロパティを持つオブジェクトである
POST リクエストによって常にトリガされます :

-   `actionType` : アクションの種類を呼び出すために指定します。例、`delete`
-   `entities` : 更新するエンティティのリストを保持しているオブジェクトの
    配列です。オペレーションを実行するため関連エンティティ・データと一緒に
    使用されます

<a name="example-crud-operations-using-fiware"></a>

# FIWARE を使用した CRUD オペレーションの例

次の例では、Orion Context Broker が、`localhost` のポート 1026 をリッスンしてい
ると仮定し、最初のシード・データを前のチュートリアルからインポートしました。

すべての例は、在庫管理システムで定義された **Product** エンティティを参照してい
ます。したがって、CRUD オペレーションは、製品または一連の製品の追加、読み取り、
修正および削除に関連します。これは、ストアの地域マネージャーの典型的な使用例です
。たとえば、価格の設定や販売可能な製品の決定です。それぞれのケースで受け取った実
際のレスポンスは、その時のシステムのコンテキスト・データの状態によって異なります
。すでに間違ってエンティティを削除した場合は、コマンドラインからデータを再ロード
して初期コンテキストを復元することができます。

```console
./import-data
```

<a name="create-operations"></a>

## 作成 (Create) オペレーション

作成 (Create) オペレーションは、HTTP POST へマップします。

-   `/v2/entities` エンドポイントは、新しいエンティティを作成するために使用され
    ます
-   `/v2/entities/<entity>` エンドポイントは、新しい属性を追加するために使用され
    ます

新しく作成されたエンティティは、`id` と `type` 属性を持たなければならず、
他の属性はオプションであり、モデル化されるシステムに依存します。
しかし、追加の属性が存在する場合、それぞれは `type` と `value` の両方を
指定する必要があります。

オペレーションが成功した場合、レスポンスは、**204 - No Context** になるか、オペ
レーションが失敗した場合、**422 - Unprocessable Entity** になります

<a name="create-a-new-data-entity"></a>

### 新しいデータ・エンティティの作成

この例では、新しい **Product** エンティティ (99 セント の "Lemonade") をコンテキ
ストに追加します。

#### :one: リクエスト :

```console
curl -iX POST \
  --url 'http://localhost:1026/v2/entities' \
  --header 'Content-Type: application/json' \
  --data ' {
      "id":"urn:ngsi-ld:Product:010", "type":"Product",
      "name":{"type":"Text", "value":"Lemonade"},
      "size":{"type":"Text", "value": "S"},
      "price":{"type":"Integer", "value": 99}
}'
```

新しいエンティティは、`/v2/entities` エンドポイントに POST リクエストを行うこと
で追加できます。

いずれかの属性がすでにコンテキストに存在する場合、リクエストは失敗します。

#### :two: リクエスト :

GET リクエストを行うことで、新しい **Product** がコンテキスト内に見つかるかどう
かを確認することができます :

```console
curl -X GET \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:010?type=Product'
```

<a name="create-a-new-attribute"></a>

### 新しい属性の作成

この例では、`id=urn:ngsi-ld:Product:001` の既存 **Product** エンティティに新しい
`specialOffer` 属性を追加します。

#### :three: リクエスト :

```console
curl -iX POST \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:001/attrs' \
  --header 'Content-Type: application/json' \
  --data '{
      "specialOffer":{"value": true}
}'
```

新しい属性は、`/v2/entities/<entity>/attrs` エンドポイントに POST リクエストを行
うことで追加できます。

ペイロードは、示されているように属性名と値を保持する JSON オブジェクトで構成する
必要があります。

`type` が指定されていない場合、デフォルト・タイプ(`Boolean`, `Text`, `Number` ま
たは `StructuredValue`)が割り当てられます。

同じ `id` を使用する後続のリクエストは、コンテキスト内の属性の値を更新します。

#### :four: リクエスト :

GET リクエストを行うことで、新しい **Product** 属性がコンテキスト内にあるかどう
かを確認することができます :

```console
curl -X GET \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:001?type=Product'
```

ご覧のように、 "Beer" **Product** エンティティにブール値の `specialOffer` フラグ
が付けられています。

<a name="batch-create-new-data-entities-or-attributes"></a>

### 新しいデータ・エンティティまたは属性のバッチ作成

この例では、コンビニエンス・バッチ処理エンドポイントを使用して、2 つの新しい
**Product** エンティティと 1 つの新しい属性 (`offerPrice`)をコンテキストに追加し
ます。

#### :five: リクエスト :

```console
curl -iX POST \
  --url 'http://localhost:1026/v2/op/update' \
  --header 'Content-Type: application/json' \
  --data '{
  "actionType":"append_strict",
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

バッチ処理は、2 つの属性のペイロードで `/v2/op/update` エンドポイントを使用しま
す

-   `actionType=append_strict` は、すべてのエンティティ/属性が新しい場合にのみ
　　リクエストが成功することを意味します
-   `entities` 属性は、作成したいエンティティの配列を保持しています。

`actionType=append_strict` バッチ・オペレーションで同じデータを使用した後続のリ
クエストでは、エラー・レスポンスが発生します。

<a name="batch-createoverwrite-new-data-entities"></a>

### 新しいデータ・エンティティのバッチ作成/上書き

この例では、コンビニエンス・バッチ処理エンドポイントを使用して、2 つの
**Product** エンティティと 1 つの属性(`offerPrice`)をコンテキストに追加または修
正します。

-   エンティティがすでに存在する場合、リクエストはエンティティの属性を更新します
-   エンティティが存在しない場合は、新しいエンティティが作成されます

#### :six: リクエスト :

```console
curl -iX POST \
  --url 'http://localhost:1026/v2/op/update' \
  --header 'Content-Type: application/json' \
  --data '{
  "actionType":"append",
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

バッチ処理では、次の 2 つの属性を持つペイロードで、`/v2/op/update` エンドポイン
トを使用します :

-   `actionType=append` は、既存のエンティティが存在する場合は上書きすることを意
    味します
-   エンティティ属性は、作成/上書きするエンティティの配列を保持します

同じデータ (すなわち、同じエンティティおよび `actionType=append` ) を含む後続の
リクエストは、コンテキスト状態を変更しません。

<a name="read-operations"></a>

## 読み取り (Read) オペレーション

-   `/v2/entities` エンドポイントは、エンティティをリストするために使用されま
    す
-   `/v2/entities/<entity>` エンドポイントは、単一のエンティティの詳細情報を取得
    するために使用されています

<a name="filtering"></a>

### フィルタリング

-   options パラメータ (attrs パラメータと組み合わせて)は、返されるフィールドを
    フィルタリングすることができます
-   q パラメータは、返されるエンティティをフィルタリングするために使用できます

<a name="read-a-data-entity"></a>

### データ・エンティティ(詳細)を読み取り

この例では、既知の `id` を持つ既存 **Product** エンティティから完全なコンテキス
トを読み取ります。

#### :seven: リクエスト :

```console
curl -X GET \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:010?type=Product'
```

#### レスポンス :

**Product** `urn:ngsi-ld:Product:010` は 99 セントの"Lemonade" です。レスポンス
は以下のようになります :

```json
{
    "id": "urn:ngsi-ld:Product:010",
    "type": "Product",
    "name": { "type": "Text", "value": "Lemonade", "metadata": {} },
    "price": { "type": "Integer", "value": 99, "metadata": {} },
    "size": { "type": "Text", "value": "S", "metadata": {} }
}
```

コンテキスト・データは、`/v2/entities/<entity>` エンドポイントに GET リクエスト
を出すことによって取り出すことができます。

<a name="read-an-attribute-from-a-data-entity"></a>

### データ・エンティティから属性の読み取り

この例では、既知の `id` を持つ既存 **Product** エンティティから単一の属性
(`name`)の値を読み取ります。

#### :eight: リクエスト :

```console
curl -X GET \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:001/attrs/name/value'
```

#### レスポンス :

**Product** `urn:ngsi-ld:Product:001` は 99 セントの"Beer"です。レスポンスは以下
のようになります :

```json
"Beer"
```

コンテキスト・データは、`/v2/entities/<entity>/attrs/<attribute>/value` エンドポ
イントに GET リクエストを出すことによって取り出すことができます。

<a name="read-a-data-entity-key-value-pairs"></a>

### データ・エンティティ(キー値のペア)の読み取り

この例では、既存の **Product** エンティティのコンテキストから2つの属性
(`name`と` price`) のキーと値のペアを既知の `id` で読み込みます。

#### :nine: リクエスト :

```console
curl -X GET \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:001?type=Product&options=keyValues&attrs=name,price'
```

#### レスポンス :

**Product** `urn:ngsi-ld:Product:001` は 99 セントの"Beer"です。レスポンスは以下
のようになります :

```json
{
    "id": "urn:ngsi-ld:Product:001",
    "type": "Product",
    "name": "Beer",
    "price": 99
}
```

`options=keyValues` パラメータと `attrs` パラメータを組み合わせて、キーと値
のペアを取得します。

<a name="read-multiple-attributes-values-from-a-data-entity"></a>

### データ・エンティティから複数の属性値の読み取り

この例では、既存の **Product** エンティティのコンテキストから、既知の ID を
持つ2つの属性 (`name`と `price`) の値を読み込みます。

#### :one::zero: リクエスト :

```console
curl -X GET \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:001?type=Product&options=values&attrs=name,price'
```

#### レスポンス :

**Product** `urn:ngsi-ld:Product:001` は 99 セントの"Beer"です。レスポンスは以下
のようになります :

```json
["Beer", 99]
```

`options=values` パラメータと `attrs` パラメータを組み合わせることで、配列内の値
のリストを返します。

<a name="list-all-data-entities-verbose"></a>

### すべてのデータ・エンティティ(詳細)を一覧表示

この例では、すべての **Product** エンティティの完全なコンテキストをリストしてい
ます。

#### :one::one: リクエスト :

```console
curl -X GET \
  --url 'http://localhost:1026/v2/entities?type=Product'
```

### レスポンス :

起動時にコンテキストが9つの製品を保持し、3つが作成オペレーションによって
追加され、フル・コンテキストに12の製品が含まれるようになりました。

```json
[
    {
        "id": "urn:ngsi-ld:Product:001",
        "type": "Product",
        "name": { "type": "Text", "value": "Beer", "metadata": {} },
        "offerPrice": { "type": "Integer", "value": 89, "metadata": {} },
        "price": { "type": "Integer", "value": 99, "metadata": {} },
        "size": { "type": "Text", "value": "S", "metadata": {} },
        "specialOffer": { "type": "Boolean", "value": true, "metadata": {} }
    },
    {
        "id": "urn:ngsi-ld:Product:002",
        "type": "Product",
        "name": { "type": "Text", "value": "Red Wine", "metadata": {} },
        "price": { "type": "Integer", "value": 1099, "metadata": {} },
        "size": { "type": "Text", "value": "M", "metadata": {} }
    },
    {
        "id": "urn:ngsi-ld:Product:003",
        "type": "Product",
        "name": { "type": "Text", "value": "White Wine", "metadata": {} },
        "price": { "type": "Integer", "value": 1499, "metadata": {} },
        "size": { "type": "Text", "value": "M", "metadata": {} }
    },
    {
        "id": "urn:ngsi-ld:Product:004",
        "type": "Product",
        "name": { "type": "Text", "value": "Vodka", "metadata": {} },
        "price": { "type": "Integer", "value": 5000, "metadata": {} },
        "size": { "type": "Text", "value": "XL", "metadata": {} }
    },
    {
        "id": "urn:ngsi-ld:Product:005",
        "type": "Product",
        "name": { "type": "Text", "value": "Lager", "metadata": {} },
        "price": { "type": "Integer", "value": 99, "metadata": {} },
        "size": { "type": "Text", "value": "S", "metadata": {} }
    },
    {
        "id": "urn:ngsi-ld:Product:006",
        "type": "Product",
        "name": { "type": "Text", "value": "Whisky", "metadata": {} },
        "price": { "type": "Integer", "value": 99, "metadata": {} },
        "size": { "type": "Text", "value": "S", "metadata": {} }
    },
    {
        "id": "urn:ngsi-ld:Product:007",
        "type": "Product",
        "name": { "type": "Text", "value": "Gin", "metadata": {} },
        "price": { "type": "Integer", "value": 99, "metadata": {} },
        "size": { "type": "Text", "value": "S", "metadata": {} }
    },
    {
        "id": "urn:ngsi-ld:Product:008",
        "type": "Product",
        "name": { "type": "Text", "value": "Apple Juice", "metadata": {} },
        "price": { "type": "Integer", "value": 99, "metadata": {} },
        "size": { "type": "Text", "value": "S", "metadata": {} }
    },
    {
        "id": "urn:ngsi-ld:Product:009",
        "type": "Product",
        "name": { "type": "Text", "value": "Orange Juice", "metadata": {} },
        "price": { "type": "Integer", "value": 99, "metadata": {} },
        "size": { "type": "Text", "value": "S", "metadata": {} }
    },
    {
        "id": "urn:ngsi-ld:Product:010",
        "type": "Product",
        "name": { "type": "Text", "value": "Lemonade", "metadata": {} },
        "price": { "type": "Integer", "value": 99, "metadata": {} },
        "size": { "type": "Text", "value": "S", "metadata": {} }
    },
    {
        "id": "urn:ngsi-ld:Product:011",
        "type": "Product",
        "name": { "type": "Text", "value": "Brandy", "metadata": {} },
        "price": { "type": "Integer", "value": 1199, "metadata": {} },
        "size": { "type": "Text", "value": "M", "metadata": {} }
    },
    {
        "id": "urn:ngsi-ld:Product:012",
        "type": "Product",
        "name": { "type": "Text", "value": "Port", "metadata": {} },
        "price": { "type": "Integer", "value": 1099, "metadata": {} },
        "size": { "type": "Text", "value": "M", "metadata": {} }
    }
]
```

<a name="list-all-data-entities-key-value-pairs"></a>

### すべてのデータ・エンティティ(キー値のペア)を一覧表示

この例では、すべての **Product** エンティティの `name` と `price` 属性を示します
。

#### :one::two: リクエスト :

```console
curl -X GET \
  --url 'http://localhost:1026/v2/entities/?type=Product&options=keyValues&attrs=name,price'
```

#### レスポンス :

起動時にコンテキストが9つの製品を保持し、3つが作成オペレーションによって
追加され、フル・コンテキストに12の製品が含まれるようになりました。

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

指定されたエンティティ・タイプのフル・コンテキスト・データは、`/v2/entities` エ
ンドポイントへの GET リクエストを行い、`type` パラメータを指定することによって取
得できます。`options=keyValues` パラメータおよび `attrs` パラメータと組み合わせ
て、キーと値を取得します。

<a name="list-data-entity-by-id"></a>

### ID でデータ・エンティティを一覧表示

この例では、すべての **Prodcut** エンティティの `id` と `type` が一覧表示されま
す。

#### :one::three: リクエスト :

```console
curl -X GET \
  --url 'http://localhost:1026/v2/entities/?type=Product&options=count&attrs=id'
```

#### レスポンス :

起動時にコンテキストが9つの製品を保持し、3つが作成オペレーションによって
追加され、フル・コンテキストに12の製品が含まれるようになりました。

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

指定されたエンティティ・タイプのコンテキスト・データは、`/v2/entities` エンドポ
イントへの GET リクエストを行い、`type` パラメータを指定することによって取得でき
ます。これと、`options=count`, `attrs=id` を組み合わせ id て指定された `type` の
`id` 属性を返します。

<a name="update-operations"></a>

## 更新 (Update) オペレーション

上書きオペレーションは HTTP PUT にマップされます。HTTP PATCH を使用すると、一度
に複数の属性を更新できます。

-   `/v2/entities/<entity>/attrs/<attribute>/value` エンドポイントは属性を更新す
    るために使用されます
-   `/v2/entities/<entity>/attrs` エンドポイントは、複数の属性を更新するために使
    用されます

<a name="overwrite-the-value-of-an-attribute-value"></a>

### 属性の値を上書き

この例では、`id=urn:ngsi-ld:Product:001` を持つ Entity の `price` 属性の値を次の
ように更新します。

#### :one::four: リクエスト :

```console
curl -iX PUT \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:001/attrs/price/value' \
  --header 'Content-Type: text/plain' \
  --data 89
```

既存の属性値は、`/v2/entities/<entity>/attrs/<attribute>/value` エンドポイントに
PUT リクエストを行うことで変更できます。

<a name="overwrite-multiple-attributes-of-a-data-entity"></a>

### データ・エンティティの複数の属性を上書き

この例は、`id=urn:ngsi-ld:Product:001` を持つ、`Entity` の `price` 属性と `name`
属性の両方の値を同時に更新します。

#### :one::five: リクエスト :

```console
curl -iX PATCH \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:001/attrs' \
  --header 'Content-Type: application/json' \
  --data ' {
      "price":{"type":"Integer", "value": 89},
      "name": {"type":"Text", "value": "Ale"}
}'
```

<a name="batch-overwrite-attributes-of-multiple-data-entities"></a>

### 複数のデータ・エンティティの属性のバッチ上書き

この例では、コンビニエンス・バッチ処理エンドポイントを使用して、
既存の製品を更新します。

#### :one::six: リクエスト :

```console
curl -iX POST \
  --url 'http://localhost:1026/v2/op/update' \
  --header 'Content-Type: application/json' \
  --data '{
  "actionType":"update",
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

バッチ処理では、2 つの属性のペイロードを持つ `/v2/op/update` エンドポイントを使
用します。

-   `actionType=append` は、既存のエンティティが存在する場合はそれを上書きするこ
    とを意味しますが、`entities` 属性は更新したいエンティティの配列を保持します
    。

<a name="batch-createoverwrite-attributes-of-multiple-data-entities"></a>

### 複数のデータ・エンティティの属性のバッチ作成/上書き

この例では、コンビニエンス・バッチ処理エンドポイントを使用して、
既存の製品を更新します。

#### :one::seven: リクエスト :

```console
curl -iX POST \
  --url 'http://localhost:1026/v2/op/update' \
  --header 'Content-Type: application/json' \
  --data '{
  "actionType":"append",
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

バッチ処理では、2 つの属性のペイロードを持つ `/v2/op/update` エンドポイントを使
用します。

-   `actionType=append` は、既存のエンティティが存在する場合はそれを上書きするこ
    とを意味しますが、`entities` 属性は更新したいエンティティの配列を保持します
    。

<a name="batch-replace-entity-data"></a>

### エンティティ・データの一括置換

この例では、既存の製品のエンティティのデータを置き換えるために、
コンビニエンス・バッチ処理エンドポイントを使用しています。

#### :one::eight: リクエスト :

```console
curl -iX POST \
  --url 'http://localhost:1026/v2/op/update' \
  --header 'Content-Type: application/json' \
  --data '{
  "actionType":"replace",
  "entities":[
    {
      "id":"urn:ngsi-ld:Product:010", "type":"Product",
      "price":{"type":"Integer", "value": 1199}
    }
  ]
}'
```

バッチ処理では、2 つの属性のペイロードを持つ `/v2/op/update` エンドポイントを使
用します。 - `actionType=replace` は、既存のエンティティが存在する場合はそれを
上書きすることを意味しますが、`entities` 属性はデータを置き換えるエンティティの
配列を保持します。

<a name="delete-operations"></a>

## 削除 (DELETE) オペレーション

削除 (Delete) オペレーション は、HTTP DELETE にマップします。

-   `/v2/entities/<entity>` エンドポイントは、エンティティを削除するために使用
    できます
-   `/v2/entities/<entity>/attrs/<attribute>` エンドポイントは、属性を削除するた
    めに使用できます

レスポンスは、オペレーションが成功すれば、**204 - No Content** となり、失敗すれ
ば、**404 - Not Found** となります。

<a name="data-relationships"></a>

### データのリレーションシップ

コンテキスト内に互いに関係するエンティティがある場合、エンティティを削除するとき
は注意が必要です。エンティティが削除されるとリファレンスが残っていないことを確認
する必要があります。

カスケード削除の構成はこのチュートリアルの範囲を超えていますが、バッチ削除リクエ
ストを使用することもできます。

<a name="delete-an-entity"></a>

### エンティティの削除

この例では、`id=urn:ngsi-ld:Product:001` のエンティティをコンテキストから削除し
ます。

#### :one::nine: リクエスト :

```console
curl -iX DELETE \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:010'
```

エンティティは、`/v2/entities/<entity>` エンドポイントに DELETE リクエストを行う
ことによって削除することができます。

同じ `id` を使用する後続のリクエストは、エンティティがもはやコンテキストに存在し
ないため、エラーレスポンスになります。

<a name="delete-an-attribute-from-an-entity"></a>

### エンティティからの属性の削除

この例では、`id=urn:ngsi-ld:Product:001` のエンティティから `specialOffer` 属性
を削除します。

#### :two::zero: リクエスト :

```console
curl -iX DELETE \
  --url 'http://localhost:1026/v2/entities/urn:ngsi-ld:Product:001/attrs/specialOffer'
```

属性は、`/v2/entities/<entity>/attrs/<attribute>` エンドポイントに DELETE リクエ
ストを行うことによって削除できます。

属性がコンテキストに存在しない場合は、エラー・レスポンスになります。

<a name="batch-delete-multiple-entities"></a>

### 複数のエンティティの一括削除

この例では、コンビニエンス・バッチ処理エンドポイントを使用して、
**Product** エンティティを削除します。

#### :two::one: リクエスト :

```console
curl -iX POST \
  --url 'http://localhost:1026/v2/op/update' \
  --header 'Content-Type: application/json' \
  --data '{
  "actionType":"delete",
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

バッチ処理では、2 つの属性のペイロードを持つ `/v2/op/update` エンドポイントを使
用します。

-   `actionType=delete` は、コンテキストから何かを削除することを意味し
    、`entities`属性は、削除するエンティティの `id` を保持します。

コンテキスト内にエンティティが存在しない場合は、
結果はエラー・レスポンスが返されます。

<a name="batch-delete-multiple-attributes-from-an-entity"></a>

### エンティティからの複数の属性のバッチ削除

この例では、コンビニエンス・バッチ処理エンドポイントを使用して、
**Product** エンティティからいくつかの属性を削除します。

#### :two::two: リクエスト :

```console
curl -iX POST \
  --url 'http://localhost:1026/v2/op/update' \
  --header 'Content-Type: application/json' \
  --data '{
  "actionType":"delete",
  "entities":[
    {
      "id":"urn:ngsi-ld:Product:003", "type":"Product",
      "price":{},
      "name": {}
    }
  ]
}'
```

バッチ処理では、2 つの属性のペイロードを持つ `/v2/op/update` エンドポイントを使
用します。

-   `actionType=delete` は、コンテキストから何かを削除することを意味し
    、`entities`属性は、削除する属性の 配列 を保持します。

コンテキストに属性が存在しない場合、結果はエラーレスポンスになります。

<a name="find-existing-data-relationships"></a>

### 既存のデータ・リレーションシップを見つける

この例では、`urn:ngsi-ld:Product:001` に直接関連するすべてのエンティティのキーを
返します。

#### :two::three: リクエスト :

```console
curl -X GET \
  --url 'http://localhost:1026/v2/entities/?q=refProduct==urn:ngsi-ld:Product:001&options=count&attrs=type'
```

#### レスポンス :

```json
[
    {
        "id": "urn:ngsi-ld:InventoryItem:001",
        "type": "InventoryItem"
    }
]
```

-   このリクエストによって空の配列が返された場合、エンティティには関連付けられて
    いません。安全に削除できます
-   レスポンスが一連の `InventoryItem` エンティティをリストする場合、関連する
    `Product` エンティティがコンテキストから削除される前に削除する必要があります

以前に、**Product** `urn:ngsi-ld:Product:001` を削除したので、上で見たものは、
実際にぶら下がている参照 (a dangling reference) です。たとえば、返された
**InventoryItem** は、もはや存在しない **Product** を参照します。

<a name="next-steps"></a>

# 次のステップ

高度な機能を追加することで、アプリケーションに複雑さを加える方法を知りたいですか
？このシリーズ
の[他のチュートリアル](https://www.letsfiware.jp/fiware-tutorials)を読むことで見
つけることができます :

---

## License

[MIT](LICENSE) © 2018-2019 FIWARE Foundation e.V.
