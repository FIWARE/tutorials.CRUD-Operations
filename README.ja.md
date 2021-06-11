[![FIWARE Banner](https://fiware.github.io/tutorials.CRUD-Operations/img/fiware.png)](https://www.fiware.org/developers)
[![NGSI LD](https://img.shields.io/badge/NGSI-LD-d6604d.svg)](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.04.02_60/gs_cim009v010402p.pdf)

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.CRUD-Operations.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)
[![JSON LD](https://img.shields.io/badge/JSON--LD-1.1-f06f38.svg)](https://w3c.github.io/json-ld-syntax/) <br/>
[![Documentation](https://img.shields.io/readthedocs/ngsi-ld-tutorials.svg)](https://ngsi-ld-tutorials.rtfd.io)

このチュートリアルでは、 **NGSI-LD** ユーザに CRUD 操作について説明します。このチュートリアルでは、
[NGSI-LD 仕様](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.04.02_60/gs_cim009v010402p.pdf)で詳しく
説明されているように、コンテキストを修正するさまざまな方法の使用例の概要を説明しています。以前のチュートリアルで定義した
温度センサ・モデルに基づいて、温度センサを表す一連のエンティティが作成、変更、削除されます。

このチュートリアルでは、全体で [cUrl](https://ec.haxx.se/) コマンドを使用していますが、
[Postman のドキュメント](https://fiware.github.io/tutorials.CRUD-Operations/ngsi-ld.html)としても利用できます。

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/cc52b59aaf5a55d04b42)

## コンテンツ

<details>
<summary><strong>詳細</strong></summary>

-   [NGSI-LD CRUD 操作](#ngsi-ld-crud-operations)
    -   [コンテキスト・エンティティ CRUD 操作](#context-entity-crud-operations)
    -   [コンテキスト・エンティティのバッチ操作](#context-entity-batch-operations)
-   [前提条件](#prerequisites)
    -   [Docker](#docker)
    -   [Cygwin](#cygwin)
-   [アーキテクチャ](#architecture)
-   [起動](#start-up)
    -   [作成操作](#create-operations)
        -   [新しいデータ・エンティティの作成](#create-a-new-data-entity)
        -   [新しい属性を作成](#create-new-attributes)
        -   [新しいデータ・エンティティまたは属性のバッチ作成](#batch-create-new-data-entities-or-attributes)
        -   [新しいデータ・エンティティのバッチ作成/上書き](#batch-createoverwrite-new-data-entities)
    -   [読み取り操作](#read-operations)
        -   [フィルタリング](#filtering)
        -   [データ・エンティティの読み取り (詳細)](#read-a-data-entity-verbose)
        -   [データ・エンティティから属性の読み取り](#read-an-attribute-from-a-data-entity)
        -   [データ・エンティティの読み取り (キー・バリューのペア)](#read-a-data-entity-key-value-pairs)
        -   [データ・エンティティからの複数属性値の読み取り](#read-multiple-attributes-values-from-a-data-entity)
        -   [すべてのデータ・エンティティの一覧表示 (詳細)](#list-all-data-entities-verbose)
        -   [すべてのデータ・エンティティの一覧表示 (キー・バリューのペア)](#list-all-data-entities-key-value-pairs)
        -   [ID によるデータ・エンティティのフィルタリング](#filter-data-entities-by-id)
    -   [更新操作](#update-operations)
        -   [属性値の値を上書き](#overwrite-the-value-of-an-attribute-value)
        -   [データ・エンティティの複数の属性を上書き](#overwrite-multiple-attributes-of-a-data-entity)
        -   [複数のデータ・エンティティの属性をバッチ更新](#batch-update-attributes-of-multiple-data-entities)
        -   [エンティティ・データのバッチ置換](#batch-replace-entity-data)
    -   [削除操作](#delete-operations)
        -   [データのリレーションシップ](#data-relationships)
        -   [エンティティを削除](#delete-an-entity)
        -   [エンティティから属性を削除](#delete-an-attribute-from-an-entity)
        -   [複数エンティティのバッチ削除](#batch-delete-multiple-entities)
        -   [エンティティから複数の属性をバッチ削除](#batch-delete-multiple-attributes-from-an-entity)
        -   [既存データのリレーションシップを見つける](#find-existing-data-relationships)
</details>

<a name="ngsi-ld-crud-operations"/>

# NGSI-LD CRUD 操作

> “Ninety-percent of everything is crud.”
>
> ― Theodore Sturgeon, Venture Science Fiction Magazine

**CRUD** 操作 (**Create**, **Read**, **Update**, **Delete**) は、永続ストレージの4つの基本機能です。**NGSI-LD** に
基づくスマート・システムの場合、**CRUD** アクションにより、開発者はシステム内のコンテキスト・データを操作できます。
すべての **CRUD** 操作は
[NGSI-LD 仕様](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.04.02_60/gs_cim009v010402p.pdf)
内で明確に定義されているため、すべての NGSI-LD 準拠の Context Broker は、同じ NGSI-LD 操作で同じインターフェースを
提供します。

このチュートリアルでは、各操作の根拠、それをいつ使用するか、およびさまざまな **CRUD** 操作を実行する方法について
説明します。**NGSI-LD** は **JSON-LD** に基づいているため、必須の各リクエストの一部として `@context` を渡します。
**CRUD** オペレーションの場合、これは通常 `Link` ヘッダとして渡されますが、これまで見てきたように、`Content-Type:
application/ld+json` の場合、リクエストのボディの一部として `@context` 属性を渡すこともできます。ただし、GET
リクエストにはボディがないため、GET リクエストの場合、ペイロード・ボディに `@context` を配置することはできません。

<a name="context-entity-crud-operations"/>

## コンテキスト・エンティティ CRUD 操作


個々のデータ・エンティティの CRUD 操作に使用される4つのエンドポイントがあります。これらは、RESTful
アプリケーション内の階層エンティティの通常のルールに従います。

`<entity-id>` がコンテキスト内でまだ認識されていない、または指定されていない操作の場合、`/ngsi-ld/v1/entities`
エンドポイントが使用されます。例として、これは新しいエンティティの作成に使用されます。

`<entity-id>` がコンテキスト内で認識されると、`/ngsi-ld/v1/entities/<entity-id>`
エンドポイントを使用して個々のデータ・エンティティを操作できます。

既知のエンティティに対する一般的な属性操作は `/ngsi-ld/v1/entities/<entity-id>/attrs` エンドポイントで行われ、
個々の属性に対する操作は `/ngsi-ld/v1/entities/<entity-id>/attrs/<attr-id>` で行われます。

データをリクエストしたり、個々のエンティティを変更したりするときに、さまざまな CRUD 操作が HTTP
動詞に自然にマッピングされます。

-   **GET** - データの読み取り
-   **POST** - 新しいエンティティと属性の作成
-   **PATCH** - エンティティと属性の修正
-   **DELETE** - エンティティと属性の削除

<a name="context-entity-batch-operations"/>

## コンテキスト・エンティティのバッチ操作

バッチ操作 (Batch operations) により、ユーザは1つのリクエストで複数のデータ・エンティティを変更できます。
すべてのバッチ操作は POST HTTP 動詞にマップされます。

-   `/entityOperations/create`
-   `/entityOperations/update`
-   `/entityOperations/upsert`
-   `/entityOperations/delete`

<a name="prerequisites"/>

# 前提条件

<a name="docker"/>

## Docker

シンプルにするために、すべてのコンポーネントは [Docker](https://www.docker.com) を使用して実行されます。 **Docker**
は、それぞれの環境に分離されたさまざまなコンポーネントを可能にするコンテナ・テクノロジです。

-   Windows に Docker をインストールするには、[こちら](https://docs.docker.com/docker-for-windows/)の指示に従ってください
-   Mac に Docker をインストールするには、[こちら](https://docs.docker.com/docker-for-mac/)の指示に従ってください
-   Linux に Docker をインストールするには、[こちら](https://docs.docker.com/install/)の手順に従ってください

**Docker Compose** は、マルチコンテナ Docker アプリケーションを定義して実行するためのツールです。
[YAMLファイル](https://raw.githubusercontent.com/Fiware/tutorials.CRUD-Operations/master/docker-compose/orion-ld.yml)
を使用して、アプリケーションに必要なサービスを設定します。これは、すべてのコンテナ・サービスを単一のコマンドで起動
できることを意味します。Docker Compose は、Docker for Windows および Docker for Mac の一部としてデフォルトでインストール
されますが、Linux ユーザは[こちら](https://docs.docker.com/compose/install/)にある手順に従う必要があります。

<a name="cygwin"/>

## Cygwin

簡単な bash スクリプトを使ってサービスを開始します。Windows ユーザは、Windows 上の Linux ディストリビューションに
似たコマンドライン機能を提供するために [cygwin](http://www.cygwin.com/) をダウンロードするべきです。

<a name="architecture"/>

# アーキテクチャ

デモ・アプリケーションは、準拠する Context Broker に対してNGSI-LD 呼び出しを送受信します。標準化された NGSI-LD
インターフェイスは複数の Context Broker で利用できるため、1つだけ選択する必要があります。たとえば、
[Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) を選択するだけです。したがって、
アプリケーションは1つの FIWARE コンポーネントのみを使用します。

現在、Orion Context Broker は、保持しているコンテキスト・データの永続性を保つためにオープンソースの
[MongoDB](https://www.mongodb.com/) テクノロジに依存しています。

データ交換の相互運用性を促進するために、NGSI-LD Context Broker は
[JSON-LD `@context` file](https://json-ld.org/spec/latest/json-ld/#the-context) を明示的に公開して、コンテキスト・
エンティティ内に保持されるデータを定義します。これは、エンティティ・タイプと属性ごとに一意の URI を定義するため、
NGSI ドメイン外の他のサービスは、データ構造の名前を選択できるようになります。すべての `@context` ファイルは
ネットワーク上で利用できる必要があります。私たちの場合、チュートリアル・アプリケーションは一連の静的ファイルをホスト
するために使用されます。

したがって、アーキテクチャは3つの要素から構成されます:

-   [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/rep/NGSI-LD/NGSI-LD/raw/master/spec/updated/generated/full_api.json)
    を使ってリクエストを受け取る [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/)
-   基礎となる [MongoDB](https://www.mongodb.com/) データベース :
    -   データ・エンティティ、サブスクリプション、レジストレーションなどのコンテキスト・データ情報を保持するために
        Orion Context Broker によって使用されます
-   **チュートリアル・アプリケーション**は次のことを行います。
    -   システム内のコンテキスト・エンティティを定義する静的な `@context` ファイルを提供します

3つの要素間のすべての対話は HTTP リクエストによって開始されるため、要素をコンテナ化して、公開されたポートから
実行できます。

![](https://fiware.github.io/tutorials.CRUD-Operations/img/architecture-ld.png)

必要な設定情報は、関連する `orion-ld.yml` ファイルの services セクションで確認できます:

```yaml
orion:
    image: fiware/orion-ld
    hostname: orion
    container_name: fiware-orion
    depends_on:
        - mongo-db
    networks:
        - default
    ports:
        - "1026:1026"
    command: -dbhost mongo-db -logLevel DEBUG
    healthcheck:
        test: curl --fail -s http://orion:1026/version || exit 1
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
    command: --nojournal
```

```yaml
tutorial:
    image: fiware/tutorials.ngsi-ld
    hostname: tutorial
    container_name: fiware-tutorial
    networks:
        default:
            aliases:
                - context
    expose:
        - 3000
```

必要な設定情報は、関連する `docker-compose.yml` ファイルの services セクションで確認できます。
[以前のチュートリアル](https://github.com/FIWARE/tutorials.Working-with-At-Context/)で説明されています。

<a name="start-up"/>

# 起動

すべてのサービスは、リポジトリ内で提供される
[services](https://github.com/FIWARE/tutorials.CRUD-Operations/blob/master/services) Bash スクリプトを実行して、
コマンドラインから初期化できます。以下のようにコマンドを実行して、リポジトリのクローンを作成して必要なイメージを
作成してください :

```bash
git clone https://github.com/FIWARE/tutorials.CRUD-Operations.git
cd tutorials.CRUD-Operations
git checkout NGSI-LD

./services orion|scorpio
```

> **注 :** クリーンアップして最初からやり直す場合は、次のコマンドで実行できます :
>
> ```
> ./services stop
> ```

---

<a name="create-operations"/>

## 作成操作

作成操作は、HTTP POST にマップされます。

-   `/ngsi-ld/v1/entities` エンドポイントは、新しいエンティティを作成するために使用されます
-   `/ngsi-ld/v1/entities/<entity-id>/attrs` エンドポイントは、新しい属性を追加するために使用されます

新しく作成されたエンティティには id, type 属性と有効な @context 定義が必要です。他のすべての属性はオプションであり、
モデル化されるシステムによって異なります。ただし、追加の属性がある場合は、それぞれに `type` と `value`
の両方を指定する必要があります。

操作が成功した場合は、レスポンスは **201 - Created** で、操作が失敗した場合は **409 - Conflict** です。

<a name="create-a-new-data-entity"/>

### 新しいデータ・エンティティの作成

この例では、新しい **TemperatureSensor** エンティティをコンテキストに追加します。

#### :one: リクエスト:

```console
curl -iX POST 'http://localhost:1026/ngsi-ld/v1/entities/' \
-H 'Content-Type: application/json' \
-H 'Link: <http://context-provider:3000/data-models/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
--data-raw '{
      "id": "urn:ngsi-ld:TemperatureSensor:001",
      "type": "TemperatureSensor",
      "category": {
            "type": "Property",
            "value": "sensor"
      },
      "temperature": {
            "type": "Property",
            "value": 25,
            "unitCode": "CEL"
      }
}'
```

`/ngsi-ld/v1/entities` エンドポイントに POST リクエストを行うことで、新しいエンティティを追加できます。

エンティティが既にコンテキストに存在する場合、リクエストは失敗します。

#### :two: リクエスト:

GET リクエストを行うことで、新しい **TemperatureSensor** がコンテキスト内で見つかるかどうかを確認できます。

```console
curl -L -X GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:TemperatureSensor:001' \
-H 'Link: <http://context-provider:3000/data-models/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"'
```

<a name="create-new-attributes"/>

### 新しい属性を作成

この例では、`id=urn:ngsi-ld:TemperatureSensor:001` を使用して、既存の **TemperatureSensor** エンティティに新しい
`batteryLevel` プロパティと `controlAsset` リレーションシップを追加します。

#### :three: リクエスト:

```console
curl -iX POST 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:TemperatureSensor:001/attrs' \
-H 'Content-Type: application/json' \
-H 'Link: <http://context-provider:3000/data-models/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
--data-raw '{
       "batteryLevel": {
            "type": "Property",
            "value": 0.9,
            "unitCode": "C62"
      },
      "controlledAsset": {
            "type": "Relationship",
            "object": "urn:ngsi-ld:Building:barn002"
      }
}'
```

`/ngsi-ld/v1/entities/<entity>/attrs`  エンドポイントに POST リクエストを行うことにより、新しい属性を追加できます。

ペイロードは、次に示すように属性名と値を保持する JSON オブジェクトで構成する必要があります。

すべての `type=Property` 属性には、関連付けられた `value` が必要です。 すべての `type=Relationship` 属性には、
別のエンティティの URN を保持する `object` が関連付けられている必要があります。`unitCode` などの明確に定義された
一般的なメタデータ要素は文字列として提供できます。他のすべてのメタデータは、独自の `type` および `value` 属性を持つ
JSON オブジェクトとして渡す必要があります。

同じもの `id` を使用する後続のリクエストは、コンテキスト内の属性の値を更新します。

#### :four: リクエスト:

GET リクエストを行うことで、新しい **TemperatureSensor** がコンテキスト内で見つかるかどうかを確認できます。

```console
curl -L -X GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:TemperatureSensor:001' \
-H 'Link: <http://context-provider:3000/data-models/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"'
```

ご覧のとおり、エンティティに追加された2つの追加属性 (`batteryLevel` と `controlledAsset`) があります。これらの属性は
**Device** モデルの一部として `@context` で定義されているため、短い名前を使用して読み取ることができます。

<a name="batch-create-new-data-entities-or-attributes"/>

### 新しいデータ・エンティティまたは属性のバッチ作成

この例では、便利なバッチ処理エンドポイントを使用して、3つの新しい **TemperatureSensor** エンティティをコンテキストに
追加します。バッチ作成は `/ngsi-ld/v1/entityOperations/create` エンドポイントを使用します。

#### :five: リクエスト:

```console
curl -iX POST 'http://localhost:1026/ngsi-ld/v1/entityOperations/create' \
-H 'Content-Type: application/json' \
-H 'Link: <http://context-provider:3000/data-models/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/json' \
--data-raw '[
    {
      "id": "urn:ngsi-ld:TemperatureSensor:002",
      "type": "TemperatureSensor",
      "category": {
            "type": "Property",
            "value": "sensor"
      },
      "temperature": {
            "type": "Property",
            "value": 20,
            "unitCode": "CEL"
      }
    },
    {
      "id": "urn:ngsi-ld:TemperatureSensor:003",
      "type": "TemperatureSensor",
      "category": {
            "type": "Property",
            "value": "sensor"
      },
      "temperature": {
            "type": "Property",
            "value": 2,
            "unitCode": "CEL"
      }
    },
     {
      "id": "urn:ngsi-ld:TemperatureSensor:004",
      "type": "TemperatureSensor",
      "category": {
            "type": "Property",
            "value": "sensor"
      },
      "temperature": {
            "type": "Property",
            "value": 100,
            "unitCode": "CEL"
      }
    }
]'
```

コンテキストに属性がすでに存在する場合、リクエストは失敗します。レスポンスでは、成功したアクションと失敗の理由
(発生した場合) が強調表示されます。

```jsonld
[
  "urn:ngsi-ld:TemperatureSensor:002",
  "urn:ngsi-ld:TemperatureSensor:003",
  "urn:ngsi-ld:TemperatureSensor:004"
]
```

<a name="batch-createoverwrite-new-data-entities"/>

### 新しいデータ・エンティティのバッチ作成/上書き

この例では、便利なバッチ処理エンドポイントを使用して、コンテキスト内の2つの **TemperatureSensor**
エンティティを追加または修正します。

-   エンティティがすでに存在する場合、リクエストはそのエンティティの属性を更新します
-   エンティティが存在しない場合は、新しいエンティティが作成されます

#### :six: リクエスト:

```console
curl -iX POST 'http://localhost:1026/ngsi-ld/v1/entityOperations/upsert' \
-H 'Content-Type: application/json' \
-H 'Link: <http://context-provider:3000/data-models/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/ld+json' \
--data-raw '[
    {
      "id": "urn:ngsi-ld:TemperatureSensor:002",
      "type": "TemperatureSensor",
      "category": {
            "type": "Property",
            "value": "sensor"
      },
      "temperature": {
            "type": "Property",
            "value": 21,
            "unitCode": "CEL"
      }
    },
    {
      "id": "urn:ngsi-ld:TemperatureSensor:003",
      "type": "TemperatureSensor",
      "category": {
            "type": "Property",
            "value": "sensor"
      },
      "temperature": {
            "type": "Property",
            "value": 27,
            "unitCode": "CEL"
      }
    }
]'
```

作成/上書きのバッチ処理は `/ngsi-ld/v1/entityOperations/upsert` エンドポイントを使用します。

同じデータ (つまり、同じエンティティと `actionType=append`) を含む後続のリクエストも成功し、
コンテキスト状態は変更されません。ただし、`modifiedAt` メタデータは修正されます。

<a name="read-operations"/>

## 読み取り操作

-   `/ngsi-ld/v1/entities` エンドポイントは、エンティティをリストするために使用されます
-   `/ngsi-ld/v1/entities/<entity>` エンドポイントは、単一のエンティティの詳細を取得するために使用されます

読み取り操作の場合は、`@context` は、`Link` ヘッダで指定する必要があります。

<a name="filtering"/>

### フィルタリング

-   `options` パラメータ(`attrs` パラメータと組み合わせる) は、返されたフィールドをフィルタリングするために使用できます
-   `q` パラメータは、返されたエンティティをフィルタリングするために使用できます

<a name="read-a-data-entity-verbose"/>

### データ・エンティティの読み取り (詳細)

この例では、既知の `id` を持つ既存の **TemperatureSensor** エンティティから完全なコンテキストを読み取ります。

#### :seven: リクエスト:

```console
curl -G -iX GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:TemperatureSensor:001' \
-H 'Link: <http://context-provider:3000/data-models/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-d 'options=sysAttrs'
```

#### レスポンス:

TemperatureSensor `urn:ngsi-ld:TemperatureSensor:001` は、正規化された NGSI-LD として返されます。`options=sysAttrs`
であるため、追加のメタデータが返されます。 デフォルトでは、ペイロード・ボディで `@context` が返されます。ただし、
`Accept:application/json` が設定されている場合、コンテント・ネゴシエーションにより移動される可能性があります。
完全なレスポンスは以下のとおりです:

```jsonld
{
    "@context": "http://context-provider:3000/data-models/ngsi-context.jsonld",
    "id": "urn:ngsi-ld:TemperatureSensor:001",
    "type": "TemperatureSensor",
    "createdAt": "2020-08-27T14:33:06Z",
    "modifiedAt": "2020-08-27T14:33:10Z",
    "category": {
        "type": "Property",
        "createdAt": "2020-08-27T14:33:06Z",
        "modifiedAt": "2020-08-27T14:33:06Z",
        "value": "sensor"
    },
    "temperature": {
        "type": "Property",
        "createdAt": "2020-08-27T14:33:06Z",
        "modifiedAt": "2020-08-27T14:33:06Z",
        "value": 25,
        "unitCode": "CEL"
    },
    "batteryLevel": {
        "value": 0.8,
        "type": "Property",
        "createdAt": "2020-08-27T14:33:10Z",
        "modifiedAt": "2020-08-27T14:33:10Z",
        "unitCode": "C62"
    },
    "controlledAsset": {
        "object": "urn:ngsi-ld:Building:barn002",
        "type": "Relationship",
        "createdAt": "2020-08-27T14:33:10Z",
        "modifiedAt": "2020-08-27T14:33:10Z"
    }
}
```

個々のコンテキスト・データ・エンティティは、`/ngsi-ld/v1/entities/<entity>` エンドポイントに GET
リクエストを行うことで取得できます。

<a name="read-an-attribute-from-a-data-entity"/>

### データ・エンティティから属性の読み取り

この例では、既知の `id` を持つ既存の **TemperatureSensor** エンティティから単一の属性 (`temperature`)
の値を読み取ります。

#### :eight: リクエスト:

```console
curl -G -iX GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:TemperatureSensor:001' \
-H 'Link: <http://context-provider:3000/data-models/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-d 'attrs=temperature'
```

#### レスポンス:

センサ `urn:ngsi-ld:TemperatureSensor:001` の読み取り値は 25°C です。レスポンスは次のとおりです:

```jsonld
{
    "@context": "http://context-provider:3000/data-models/ngsi-context.jsonld",
    "id": "urn:ngsi-ld:TemperatureSensor:001",
    "type": "TemperatureSensor",
    "temperature": {
        "type": "Property",
        "value": 25,
        "unitCode": "CEL"
    }
}
```

`options=keyValues` が使用されなかったため、これは `unitCode` などのメタデータを含む正規化されたレスポンスです。
コンテキスト。データは、`/ngsi-ld/v1/entities/<entity-id>` エンドポイントに GET リクエストを行い、カンマ区切りの
リストを使用して `attrs` を選択することで取得できます。

<a name="read-a-data-entity-key-value-pairs"/>

### データ・エンティティの読み取り (キー・バリューのペア)

この例では、既知の `id` を持つ既存の **TemperatureSensor** エンティティのコンテキストからキーと値のペアを読み取ります。

#### :nine: リクエスト:

```console
curl -G -iX GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:TemperatureSensor:001' \
-H 'Link: <http://context-provider:3000/data-models/json-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/json' \
-d 'options=keyValues'
```

#### レスポンス:

センサ `urn:ngsi-ld:TemperatureSensor:001` の読み取り値は25°Cです。レスポンスは次のとおりです:

```json
{
    "id": "urn:ngsi-ld:TemperatureSensor:001",
    "type": "TemperatureSensor",
    "category": "sensor",
    "temperature": 25,
    "batteryLevel": 0.8,
    "controlledAsset": "urn:ngsi-ld:Building:barn002"
}
```

レスポンスには、`urn:ngsi-ld:TemperatureSensor:001` のすべての属性を含むエンティティからのコンテキスト・データの
フィルタリングされていないリストが含まれています。`Accept: application/json` が設定されているため、ペイロード本体には
`@context` 属性が含まれていません。

`attrs` パラメータと `options=keyValues` パラメータを組み合わせて、キーと値のペアの限られたセットを取得します。

<a name="read-multiple-attributes-values-from-a-data-entity"/>

### データ・エンティティからの複数属性値の読み取り

この例では、既知の `id` を持つ既存の **TemperatureSensor** エンティティのコンテキストから2つの属性 (`category` および
`temperature`) の値を読み取ります。

#### :one::zero: リクエスト:

```console
curl -G -iX GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:TemperatureSensor:001' \
-H 'Link: <http://context-provider:3000/data-models/json-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/json' \
-d 'options=keyValues' \
-d 'attrs=category,temperature'
```

#### レスポンス:

センサ `urn:ngsi-ld:TemperatureSensor:001` の読み取り値は25°Cです。レスポンスは次のとおりです:

```json
{
    "id": "urn:ngsi-ld:TemperatureSensor:001",
    "type": "TemperatureSensor",
    "category": "sensor",
    "temperature": 25
}
```

`options=keyValues` パラメータと `attrs` パラメータを組み合わせて、値のリストを返します。

<a name="list-all-data-entities-verbose"/>

### すべてのデータ・エンティティの一覧表示 (詳細)


この例では、すべての **TemperatureSensor** エンティティの完全なコンテキストを一覧表示します。

#### :one::one: リクエスト:

```console
curl -G -iX GET 'http://localhost:1026/ngsi-ld/v1/entities/' \
-H 'Link: <http://context-provider:3000/data-models/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-d 'type=TemperatureSensor'
```

#### レスポンス:

起動時にコンテキストは空で、4つの **TemperatureSensor** エンティティが作成操作によって追加されているため、
完全なコンテキストには4つのセンサが含まれています。

```jsonld
[
    {
        "@context": "http://context-provider:3000/data-models/ngsi-context.jsonld",
        "id": "urn:ngsi-ld:TemperatureSensor:004",
        "type": "TemperatureSensor",
        "category": {
            "type": "Property",
            "value": "sensor"
        },
        "temperature": {
            "type": "Property",
            "value": 100,
            "unitCode": "CEL"
        }
    },
    {
        "@context": "http://context-provider:3000/data-models/ngsi-context.jsonld",
        "id": "urn:ngsi-ld:TemperatureSensor:002",
        "type": "TemperatureSensor",
        "category": {
            "type": "Property",
            "value": "sensor"
        },
        "temperature": {
            "type": "Property",
            "value": 21,
            "unitCode": "CEL"
        }
    },
    {
        "@context": "http://context-provider:3000/data-models/ngsi-context.jsonld",
        "id": "urn:ngsi-ld:TemperatureSensor:003",
        "type": "TemperatureSensor",
        "category": {
            "type": "Property",
            "value": "sensor"
        },
        "temperature": {
            "type": "Property",
            "value": 27,
            "unitCode": "CEL"
        }
    },
    {
        "@context": "http://context-provider:3000/data-models/ngsi-context.jsonld",
        "id": "urn:ngsi-ld:TemperatureSensor:001",
        "type": "TemperatureSensor",
        "batteryLevel": {
            "type": "Property",
            "value": 0.8,
            "unitCode": "C62"
        },
        "category": {
            "type": "Property",
            "value": "sensor"
        },
        "controlledAsset": {
            "type": "Relationship",
            "object": "urn:ngsi-ld:Building:barn002"
        },
        "temperature": {
            "type": "Property",
            "value": 25,
            "unitCode": "CEL"
        }
    }
]
```

<a name="list-all-data-entities-key-value-pairs"/>

### すべてのデータ・エンティティの一覧表示 (キー・バリューのペア)

この例では、すべての **TemperatureSensor** エンティティの `temperature` 属性を一覧表示します。

#### :one::two: リクエスト:

```console
curl -G -iX GET 'http://localhost:1026/ngsi-ld/v1/entities/' \
-H 'Link: <http://context-provider:3000/data-models/json-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/json' \
-d 'type=TemperatureSensor' \
-d 'options=keyValues' \
-d 'attrs=temperature'
```

#### レスポンス:

完全なコンテキストには4つのセンサが含まれており、ランダムな順序で返されます:

```json
[
    {
        "id": "urn:ngsi-ld:TemperatureSensor:004",
        "type": "TemperatureSensor",
        "temperature": 100
    },
    {
        "id": "urn:ngsi-ld:TemperatureSensor:002",
        "type": "TemperatureSensor",
        "temperature": 21
    },
    {
        "id": "urn:ngsi-ld:TemperatureSensor:003",
        "type": "TemperatureSensor",
        "temperature": 27
    },
    {
        "id": "urn:ngsi-ld:TemperatureSensor:001",
        "type": "TemperatureSensor",
        "temperature": 25
    }
]
```

指定されたエンティティ・タイプの完全なコンテキスト・データは、`/ngsi-ld/v1/entities/` エンドポイントに GET リクエストを行い、
`type` パラメータを指定し、これに キー・バリューを取得するための `options=keyValues` パラメータと `attrs`
パラメータを組み合わせています。

<a name="filter-data-entities-by-id"/>

### ID によるデータ・エンティティのフィルタリング

この例では、`id` によって選択された2つの **TemperatureSensor** エンティティから選択されたデータを一覧表示します。すべての
`id` は一意である必要があるため、このリクエストでは `type` は必要ありません。`id` でフィルタリングするには、
エントリをカンマ区切りのリストに追加します。

#### :one::three: リクエスト:

```console
curl -G -iX GET 'http://localhost:1026/ngsi-ld/v1/entities/'' \
-H 'Link: <http://context-provider:3000/data-models/json-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/json' \
-d 'id=urn:ngsi-ld:TemperatureSensor:001,urn:ngsi-ld:TemperatureSensor:002' \
-d 'options=keyValues' \
-d 'attrs=temperature'
```

#### レスポンス:

レスポンスは、選択されたエンティティから選択された属性の詳細を示します。

```json
[
    {
        "id": "urn:ngsi-ld:TemperatureSensor:002",
        "type": "TemperatureSensor",
        "temperature": 21
    },
    {
        "id": "urn:ngsi-ld:TemperatureSensor:001",
        "type": "TemperatureSensor",
        "temperature": 25
    }
]
```

<a name="update-operations"/>

## 更新操作

上書き操作は HTTP PATCH にマップされます:

-   `/ngsi-ld/v1/entities/<entity-id>/attrs/<attribute>` エンドポイントは属性を更新するために使用されます
-   `/ngsi-ld/v1/entities/<entity-id>/attrs` エンドポイントは、複数の属性を更新するために使用されます

<a name="overwrite-the-value-of-an-attribute-value"/>

### 属性値の値を上書き

この例では、`id=urn:ngsi-ld:TemperatureSensor:001` の id を持つエンティティの `category` 属性の値を更新します。

#### :one::four: リクエスト:

```console
curl -iX PATCH 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:TemperatureSensor:001/attrs/category' \
-H 'Content-Type: application/json' \
-H 'Link: <http://context-provider:3000/data-models/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
--data-raw '{
    "value": ["sensor", "actuator"],
    "type": "Property"
}'
```

`/ngsi-ld/v1/entities/<entity-id>/attrs/<attribute>` エンドポイントに PATCH リクエストを行うことで、既存の属性値を変更できます。
適切な `@context` を `Link` ヘッダとして提供する必要があります。

<a name="overwrite-multiple-attributes-of-a-data-entity"/>

### データ・エンティティの複数の属性を上書き

この例では、`id=urn:ngsi-ld:TemperatureSensor:001` の id を持つエンティティの `category` と `controlledAsset`
の両方の値を同時に更新します。

#### :one::five: リクエスト:

```console
curl -iX PATCH 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:TemperatureSensor:001/attrs' \
-H 'Content-Type: application/json' \
-H 'Link: <http://context-provider:3000/data-models/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
--data-raw '{
      "category": {
            "value": [
                  "sensor",
                  "actuator"
            ],
            "type": "Property"
      },
      "controlledAsset": {
            "type": "Relationship",
            "object": "urn:ngsi-ld:Building:barn001"
      }
}'
```

<a name="batch-update-attributes-of-multiple-data-entities"/>

### 複数のデータ・エンティティの属性をバッチ更新

この例では、便利なバッチ処理エンドポイントを使用して、既存のセンサを更新します。

#### :one::six: リクエスト:

```console
curl -iX POST 'http://localhost:1026/ngsi-ld/v1/entityOperations/upsert?options=update' \
-H 'Content-Type: application/json' \
-H 'Link: <http://context-provider:3000/data-models/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
--data-raw '[
  {
    "id": "urn:ngsi-ld:TemperatureSensor:003",
    "type": "TemperatureSensor",
    "category": {
      "type": "Property",
      "value": [
        "actuator",
        "sensor"
      ]
    }
  },
  {
    "id": "urn:ngsi-ld:TemperatureSensor:004",
    "type": "TemperatureSensor",
    "category": {
      "type": "Property",
      "value": [
        "actuator",
        "sensor"
      ]
    }
  }
]'
```

バッチ処理は `/ngsi-ld/v1/entityOperations/upsert` エンドポイントを使用します。ペイロードのボディには、
更新するエンティティと属性の配列が保持されます。この `options=update` パラメータは、既存の属性がすでに存在し、
ペイロードに含まれていない場合は削除しないことを示します。

代替方法は、`/ngsi-ld/v1/entityOperations/update` エンドポイントを使用することです。`upsert` とは異なり、
`update` 操作は新しいエンティティを暗黙的に作成しません。エンティティがまだ存在しない場合は失敗します。

<a name="batch-replace-entity-data"/>

### エンティティ・データのバッチ置換

この例では、便利なバッチ処理エンドポイントを使用して、既存のセンサのエンティティ・データを置き換えます。

#### :one::seven: リクエスト:

```console
curl -iX POST 'http://localhost:1026/ngsi-ld/v1/entityOperations/update?options=replace' \
-H 'Content-Type: application/json' \
-H 'Link: <http://context-provider:3000/data-models/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
--data-raw '[
  {
    "id": "urn:ngsi-ld:TemperatureSensor:003",
    "type": "TemperatureSensor",
    "category": {
      "type": "Property",
      "value": [
        "actuator",
        "sensor"
      ]
    }
  },
  {
    "id": "urn:ngsi-ld:TemperatureSensor:004",
    "type": "TemperatureSensor",
    "temperature": {
      "type": "Property",
      "value": [
        "actuator",
        "sensor"
      ]
    }
  }
]'
```

バッチ処理では、`options=replace` パラメータを指定したペイロードを持つ `/ngsi-ld/v1/entityOperations/update`
エンドポイントを使用します。これは、既存のエンティティを上書きすることを意味します。`/ngsi-ld/v1/entityOperations/upsert`
は、新しいエンティティも作成する場合にも使用できます。

<a name="delete-operations"/>

## 削除操作

削除操作は HTTP DELETE にマップされます。

-   `/ngsi-ld/v1/entities/<entity-id>` エンドポイントは、エンティティを削除するために使用できます
-   `/ngsi-ld/v1/entities/<entity-id>/attrs/<attribute>` エンドポイントは、属性を削除するために使用できます

レスポンスは、操作が成功した場合は **204 - No Content** で、操作が失敗した場合は **404 - Not Found** です。

<a name="data-relationships"/>

### データのリレーションシップ

コンテキスト内に互いに関連するエンティティがある場合は、エンティティを削除するときに注意する必要があります。
エンティティが削除されたら、参照がぶら下がっていないことを確認する必要があります。

削除のカスケードを整理することは、このチュートリアルの範囲外ですが、バッチ削除リクエストを使用して可能です。

<a name="delete-an-entity"/>

### エンティティを削除

この例では、コンテキストから `id=urn:ngsi-ld:TemperatureSensor:004` の id を持つエンティティを削除します。

#### :one::eight: リクエスト:

```console
curl -iX DELETE 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:TemperatureSensor:004'
```

エンティティは、`/ngsi-ld/v1/entities/<entity>` エンドポイントに DELETE リクエストを行うことで削除できます。

同じ `id` エンティティを使用する後続のリクエストでは、エンティティがコンテキストに存在しないため、
エラー・レスポンスが返されます。

<a name="delete-an-attribute-from-an-entity"/>

### エンティティから属性を削除

この例では、`id=urn:ngsi-ld:TemperatureSensor:001` の id を持つエンティティから `batteryLevel` 属性を削除します。

#### :one::nine: リクエスト:

```console
curl -L -X DELETE 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:TemperatureSensor:001/attrs/batteryLevel' \
-H 'Link: <http://context-provider:3000/data-models/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"'
```

`/ngsi-ld/v1/entities/<entity>/attrs/<attribute>` エンドポイントに DELETE リクエストを行うことで、属性を削除できます。
属性名を確実に認識できるようにするには、リクエストに適切な `@context` を `Link` ヘッダの形式で指定することが重要です。

エンティティがコンテキスト内に存在しないか、エンティティで属性が見つからない場合、結果はエラー・レスポンスになります。

<a name="batch-delete-multiple-entities"/>

### 複数エンティティのバッチ削除

この例では、便利なバッチ処理エンドポイントを使用して、いくつかの **TemperatureSensor** エンティティを削除します。

#### :two::zero: リクエスト:

```console
curl -L -X POST 'http://localhost:1026/ngsi-ld/v1/entityOperations/delete' \
-H 'Content-Type: application/json' \
--data-raw '[
  "urn:ngsi-ld:TemperatureSensor:002",
  "urn:ngsi-ld:TemperatureSensor:003"
]'
```

バッチ処理では、削除する要素の配列で構成されるペイロードで、 `/ngsi-ld/v1/entityOperations/delete`
エンドポイントを使用します。

エンティティがコンテキストに存在しない場合、結果はエラー・レスポンスになります。

<a name="batch-delete-multiple-attributes-from-an-entity"/>

### エンティティから複数の属性をバッチ削除

この例では、PATCH `/ngsi-ld/v1/entities/<entity-id>/attrs` エンドポイントを使用して、**TemperatureSensor**
エンティティから一部の属性を削除します。

#### :two::one: リクエスト:

```console
curl -L -X PATCH 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:TemperatureSensor:001/attrs' \
-H 'Content-Type: application/json' \
-H 'Link: <http://context-provider:3000/data-models/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
--data-raw '{
      "category": {
            "value": null,
            "type": "Property"
      },
      "controlledAsset": {
            "type": "Relationship",
            "object": null
      }
}'
```

値に `null` が設定されている場合は、属性は削除されます。

<a name="find-existing-data-relationships"/>

### 既存データのリレーションシップを見つける

この例では、リンクされたデータのリレーションシップがエンティティ `urn:ngsi-ld:Building:barn002`
に対して残っているかどうかを示すヘッダを返します。

#### :two::two: リクエスト:

```console
curl -iX GET 'http://localhost:1026/ngsi-ld/v1/entities/?type=TemperatureSensor&limit=0&count=true&q=controlledAsset==%22urn:ngsi-ld:Building:barn002%22' \
-H 'Link: <http://context-provider:3000/data-models/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/json'
```

#### レスポンス:

```json
[]
```

`limit=0` パラメータが使用されているため、ペイロード・ボディにエンティティがリストされていませんが、`count=true`
は、カウントは代わりにヘッダとして渡されれることを示します:

```text
NGSILD-Results-Count: 1
```

`limit` 存在しない場合、ペイロードは一致するすべてのエンティティの詳細を代わりに保持します。

# 次のステップ

高度な機能を追加することで、アプリケーションに複雑さを加える方法を知りたいですか？ このシリーズの
[他のチュートリアル](https://www.letsfiware.jp/ngsi-ld-tutorials)を読むことで見つけることができます

---

## License

[MIT](LICENSE) © 2020-2021 FIWARE Foundation e.V.
