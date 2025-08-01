# IoT Agents - JSON [<img src="https://img.shields.io/badge/NGSI-LD-d6604d.svg" width="90"  align="left" />](https://cim.etsi.org/NGSI-LD/official/0--1.html)[<img src="https://fiware.github.io/tutorials.IoT-Agent-JSON/img/fiware.png" align="left" width="162">](https://www.fiware.org/)<br/>

[![FIWARE IoT Agents](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/iot-agents.svg)](https://github.com/FIWARE/catalogue/blob/master/iot-agents/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Iot-Agent.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)
[![JSON LD](https://img.shields.io/badge/JSON--LD-1.1-f06f38.svg)](https://w3c.github.io/json-ld-syntax/)
[![JSON](https://img.shields.io/badge/Payload-JSON-f06f38.svg)](https://fiware-iotagent-json.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
<br/> [![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)

このチュートリアルでは、
[JSON 用の IoT Agent](https://fiware-iotagent-json.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
を使用して、ダミーの [JSON](https://json.org/) ベースの IoT デバイスを接続し、
[Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) などの NGSI-LD 準拠の Context Broker
に送信された [NGSI-LD](https://cim.etsi.org/NGSI-LD/official/0--1.html) リクエストを使用して、
測定値を読み取ったり、コマンドを送信したりできるようにします。

チュートリアルでは [cUrl](https://ec.haxx.se/) コマンドを使用しますが、
[Postman documentation](https://fiware.github.io/tutorials.IoT-Agent-JSON/ngsi-ld.html)
としても利用できます。

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/c624b462f449c58d182b)
[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/FIWARE/tutorials.IoT-Agent-JSON/tree/NGSI-LD)

## コンテンツ

<details>
<summary><strong>詳細</strong></summary>

-   [なぜ複数の IoT Agent が必要なのですか？](#what-is-an-iot-agent)
    -   [サウスバウンド・トラフィック (コマンド)](#southbound-traffic-commands)
    -   [ノースバウンド・トラフィック (測定値)](#northbound-traffic-measurements)
    -   [共通機能](#common-functionality)
-   [アーキテクチャ](#architecture)
    -   [ダミー IoT デバイスの設定](#dummy-iot-devices-configuration)
    -   [IoT Agent for JSON  の設定](#iot-agent-for-json-configuration)
-   [前提条件](#prerequisites)
    -   [Docker](#docker)
    -   [WSL](#wsl)
-   [起動](#start-up)
-   [IoT Agent のプロビジョニング](#provisioning-an-iot-agent)
    -   [IoT Agent サービスの健全性の確認](#checking-the-iot-agent-service-health)
    -   [IoT デバイスの接続](#connecting-iot-devices)
        -   [サービス・グループのプロビジョニング](#provisioning-a-service-group)
        -   [センサのプロビジョニング](#provisioning-a-sensor)
        -   [アクチュエータのプロビジョニング](#provisioning-an-actuator)
        -   [Filling Station のプロビジョニング](#provisioning-a-filling-station)
        -   [Tractor FMIS System のプロビジョニング](#provisioning-a-tractor-fmis-system)
    -   [Context Broker コマンドの有効化](#enabling-context-broker-commands)
        -   [Irrigation System の有効化](#activating-the-irrigation-system)
        -   [Tractor の有効化](#activating-the-tractor)
        -   [Filling Station の有効化](#activating-the-filling-station)
-   [サービス・グループ CRUD アクション](#service-group-crud-actions)
    -   [サービス・グループの作成](#creating-a-service-group)
    -   [サービス・グループの詳細の読み取り](#read-service-group-details)
    -   [すべてのサービス・グループのリスト](#list-all-service-groups)
    -   [サービス・グループの更新](#update-a-service-group)
    -   [サービス・グループの削除](#delete-a-service-group)
-   [デバイス CRUD アクション](#device-crud-actions)
    -   [プロビジョニングされたデバイスの作成](#creating-a-provisioned-device)
    -   [プロビジョニングされたデバイスの詳細の読み取り](#read-provisioned-device-details)
    -   [プロビジョニングされたすべてのデバイスのリスト](#list-all-provisioned-devices)
    -   [プロビジョニングされたデバイスの更新](#update-a-provisioned-device)
    -   [プロビジョニングされたデバイスの削除](#delete-a-provisioned-device)

</details>

<a name="what-is-an-iot-agent"></a>

# なぜ複数の IoT Agent が必要なのですか？

> "Ils en conclurent que la syntaxe est une fantaisie et la grammaire une illusion."
>
> — Gustave Flaubert (Bouvard and Pecuchet)

IoT Agent は、デバイスのグループが独自のネイティブ・プロトコルを使用してデータ を Context Broker に送信し、そこから管理
できるようにするコンポーネントです。すべての IoT Agent は単一のペイロード形式に対して定義されていますが、そのペイロード
に対して複数の異なるトランスポートを使用できます。。

Ultralight IoT Agent にすでに紹介しました。これは、キーと値のペアの単純なバー (`|`) で区切られたリストを使用して通信
します。このペイロードは単純で簡潔ですが、比較的あいまいな通信メカニズムです。インターネットで使用される最も一般的な
メッセージング・ペイロードは、JavaScript 開発者に馴染みのある、いわゆる JavaScript Object Notation または JSON です。

JSON は Ultralight よりも若干冗長ですが、より大きなメッセージを送信するコストは、構文の親しみやすさによって相殺されます。
多くの一般的なデバイスは JSON 形式でメッセージを送信するようにプログラムでき、データを解析するためのソフトウェア・
ライブラリが多数存在するため、個別の
[IoT Agent for JSON](https://fiware-iotagent-json.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
はこの形式で送信されるメッセージに対処するために特別に作成されました。

JSON ペイロードを使用した通信と Ultralight プレーン・テキスト・ペイロードを使用した通信の間に実際的な違いはありません。
その通信の基礎、言い換えれば、コンポーネント間でメッセージが渡される方法を定義する基本的なプロトコルは同じままです。
明らかに、IoT Agent 内の JSON ペイロードの解析、つまり、JSON から NGSI へのメッセージの変換、またはその逆変換は、JSON IoT
Agent に固有です。

2つの IoT Agent の直接比較を以下に示します:

| IoT Agent for Ultralight                                                   | IoT Agent for JSON                                                         | 関連するプロトコル領域     |
| -------------------------------------------------------------------------- | -------------------------------------------------------------------------- | -------------------------- |
| サンプル測定値 `c\|1`                                                      | サンプル測定値 `{"count": "1"}`                                            | メッセージ・ペイロード     |
| サンプル・コマンド `Robot1@turn\|left`                                     | サンプル・コマンド `{"Robot1": {"turn": "left"}}`                          | メッセージ・ペイロード     |
| コンテンツ・タイプは `text/plain`                                          | コンテンツ・タイプは `application/json`                                    | メッセージ・ペイロード     |
| 3つのトランスポートを提供 - HTTP, MQTT, AMPQ                               | 3つのトランスポートを提供 - HTTP, MQTT, AMPQ                               | トランスポート・メカニズム |
| HTTP はデフォルトで `iot/json` の測定をリッスン                               | HTTP はデフォルトで `iot/json` の測定をリッスン                            | トランスポート・メカニズム |
| HTTP デバイスはパラメータ `?i=XXX&k=YYY` で識別                            | HTTP デバイスはパラメータ `?i=XXX&k=YYY` で識別                            | デバイスの識別             |
| 既知の URL にポストされた HTTP コマンド - レスポンスはリプライに含まれます | 既知の URL にポストされた HTTP コマンド - レスポンスはリプライに含まれます | 通信ハンドシェイク         |
| MQTT デバイスは、トピック `/XXX/YYY` のパスで識別されます                  | MQTT デバイスは、トピック  `/XXX/YYY` のパスで識別されます                 | デバイスの識別             |
| `cmd` トピックに投稿された MQTT コマンド                                   | `cmd` トピックに投稿された MQTT コマンド                                   | 通信ハンドシェイク         |
| `cmdexe` トピックに投稿された MQTT コマンド・レスポンス                    | `cmdexe` トピックに投稿された MQTT コマンド                                | 通信ハンドシェイク         |

ご覧のように、メッセージ・ペイロードは2つの IoT Agent 間で完全に異なりますが、
プロトコルの残りの多くは同じままです。

<a name="southbound-traffic-commands"></a>

## サウスバウンド・トラフィック (コマンド)

Orion Context Broker によって生成され、IoT Agent 経由で、IoT デバイスに対して渡される HTTP リクエストは、サウスバウンド・
トラフィックと呼ばれます。サウスバウンド・トラフィックは、アクションによって実世界の状態を変更するアクチュエータ・
デバイスに対して行われる**コマンド**で構成されます。

たとえば、実際の JSON **Irrigation System** を有効にするには、次の相互作用が発生します。

1.  NGSI PATCH リクエストが **Context Broker** に送信され、**Irrigation System** の現在のコンテキストが更新されます

-   これは事実上、**Irrigation System** の `on` コマンドを呼び出す間接的なリクエストです

2.  **Context Broker** は、コンテキスト内のエンティティを見つけ、この属性のコンテキスト・プロビジョニングが IoT Agent
    に委任されたことを確認します
3.  標準のフォワーディング・メカニズムを使用して、**Context Broker**は PATCH リクエストを複製し、それを **IoT Agent**
    のノースポートに転送してコマンドを呼び出します
4.  **IoT Agent** は、このサウスバウンド・リクエストを受信し、それを JSON 構文に変換して、**Irrigation System** に渡します
5.  **Irrigation System** はスプリンクラーのスイッチをオンにし、コマンドの結果を JSON 構文で **IoT Agent** に返します
6.  **IoT Agent** は、このノースバウンド・リクエストを受信し、解釈し、**Context Broker** に NGSI-LD リクエストを送信する
    ことにより、相互作用の結果をコンテキストに渡します
7.  **Context Broker** はこのノースバウンド・リクエストを受信し、コマンドの結果でコンテキストを更新します

![](https://fiware.github.io/tutorials.IoT-Agent-JSON/img/command-swimlane.png)

-   **ユーザ** と **Context Broker** 間のリクエストは NGSI-LD を使用します
-   **Context Broker** と**IoT Agent** 間のリクエストは NGSI-LD を使用します
-   **IoT Agent** と**IoT デバイス** 間のリクエストはネイティブ・プロトコルを使用します
-   **IoT デバイス**と**IoT Agent** 間のリクエストはネイティブ・プロトコルを使用します
-   **IoT Agent** と **Context Broker** の間のリクエストは NGSI-LD を使用します

<a name="northbound-traffic-measurements"></a>

## ノースバウンド・トラフィック (測定値)

IoT デバイスから生成され、IoT Agent を介して、Context Broker に返送されるリクエストは、ノースバウンド・トラフィックと
呼ばれます。ノースバウンド・トラフィックは、センサ・デバイスによって行われた**測定値**で構成され、実世界の状態を
システムのコンテキスト・データに中継します。

たとえば、実際の **Soil Sensor** が湿度の読み取り値を送信する場合、次の相互作用が発生します。

1.  **Soil Sensor**は測定を行い、結果を **IoT Agent** に渡します
2.  **IoT Agent** は、このノースバウンド・リクエストを受信し、結果を JSON 構文から変換し、**Context Broker** に NGSI-LD
    リクエストを送信することにより、相互作用の結果をコンテキストに渡します
3.  **Context Broker** は、このノースバウンド・リクエストを受信し、測定結果でコンテキストを更新します

![](https://fiware.github.io/tutorials.IoT-Agent-JSON/img/measurement-swimlane.png)

-   **IoT-Device** と**IoT-Agent** 間のリクエストはネイティブ・プロトコルを使用します
-   **IoT-Agent** と**Context-Broker** 間のリクエストは NGSI-LD を使用します

> **注** より複雑な対話も可能ですが、この概要は IoT Agent の基本原則を理解するのに十分です

<a name="common-functionality"></a>

## 共通機能

以前のセクションからわかるように、各 IoT Agent は異なるプロトコルを解釈するので一意ですが、IoT Agent 間にはかなりの類似性が
あります。

-   デバイスの更新をリッスンする標準のエンドポイントを提供する
-   コンテキスト・データの更新をリッスンする標準のエンドポイントを提供する
-   デバイスのリストを保持し、コンテキスト・データ属性をデバイス構文にマッピングする
-   セキュリティ認証

この基本機能は、一般的な [IoT Agentフレームワークライブラリ](https://iotagent-node-lib.readthedocs.io/)
によって抽象化されています。

#### デバイス・モニタ

このチュートリアルの目的のために、一連のダミーの農業用 IoT デバイスが作成され、Context Broker に接続されます。 使用される
アーキテクチャとプロトコルの詳細は、[IoT センサ チュートリアル](https://github.com/FIWARE/tutorials.IoT-Sensors/tree/NGSI-LD)
に記載されています。各デバイスの状態は、次の場所にある JSON デバイス・モニタの Web ページで確認できます:
`http://localhost:3000/device/monitor`

![FIWARE Monitor](https://fiware.github.io/tutorials.IoT-Agent-JSON/img/farm-devices.png)

<a name="architecture"></a>

# アーキテクチャ

このアプリケーションは、[以前のチュートリアル](https://github.com/FIWARE/tutorials.Subscriptions/)で作成されたコンポーネントに
基づいて構築されています。[Orion](https://fiware-orion.readthedocs.io/en/latest/) などの NGSI-LD Context Broker と
[IoT Agent for JSON](https://fiware-iotagent-json.readthedocs.io/en/latest/) の2つの FIWARE コンポーネントを利用します。
アプリケーションが _“Powered by FIWARE”_ として認定されるには、Context Broker を使用するだけで十分です。 Orion Context Broker
と IoT Agent はどちらも、保持している情報の永続性を維持するためにオープンソース[MongoDB](https://www.mongodb.com/) テクノロジ
に依存しています。[以前のチュートリアル](https://github.com/FIWARE/tutorials.IoT-Sensors/) で作成されたダミー IoT
デバイスも使用します。

したがって、全体的なアーキテクチャは次の要素で構成されます:

-   [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) は、
    [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/rep/NGSI-LD/NGSI-LD/raw/master/spec/updated/generated/full_api.json)
    を使用してリクエストを受信します
-   FIWARE [IoT Agent for JSON](https://fiware-iotagent-json.readthedocs.io/en/latest/) は、
    [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/rep/NGSI-LD/NGSI-LD/raw/master/spec/updated/generated/full_api.json)
    を使用してノースバウンド・リクエストを受信し、それらをデバイスの JSON コマンドに変換します
-   基礎となる [MongoDB](https://www.mongodb.com/) データベース:
    -   **Orion Context Broker** がデータ・エンティティ、サブスクリプション、レジストレーションなどのコンテキスト・
        データ情報を保持するために使用します
    -   **IoT Agent** がデバイスの URL やキーなどのデバイス情報を保持するために使用します
-   HTTP **Web-Server** システム内のコンテキスト・エンティティを定義する静的な `@context` ファイルを提供します
-   **Tutorial Application** は次のことを行います:
    -   HTTP 上で実行される [JSON](https://fiware-iotagent-json.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
        プロトコルを使用して、ダミー[農業用 IoT デバイス](https://github.com/FIWARE/tutorials.IoT-Sensors/tree/NGSI-LD)
        のセットとして機能します

要素間の相互作用はすべて HTTP リクエストによって開始されるため、エンティティはコンテナ化され、公開されたポートから
実行できます。

![](https://fiware.github.io/tutorials.IoT-Agent-JSON/img/architecture-ld.png)

IoT デバイスと IoT Agent を接続するために必要な設定情報は、関連する `docker-compose.yml` ファイルのサービス・
セクションで確認できます。

<a name="dummy-iot-devices-configuration"></a>

## ダミー IoT デバイスの設定

```yaml
tutorial:
    image: quay.io/fiware/tutorials.ngsi-ld
    hostname: iot-sensors
    container_name: fiware-tutorial
    networks:
        - default
    expose:
        - "3000"
        - "3001"
    ports:
        - "3000:3000"
        - "3001:3001"
    environment:
        - "DEBUG=tutorial:*"
        - "PORT=3000"
        - "IOTA_HTTP_HOST=iot-agent"
        - "IOTA_HTTP_PORT=7896"
        - "DUMMY_DEVICES_PORT=3001"
        - "DUMMY_DEVICES_API_KEY=4jggokgpepnvsb2uv4s40d59ov"
        - "DUMMY_DEVICES_TRANSPORT=HTTP"
        - "DUMMY_DEVICES_PAYLOAD=JSON"
        - "IOTA_JSON_LD_CONTEXT=http://context/user-context.jsonld"
```

`tutorial` コンテナは2つのポートでリッスンしています:

-   ポート `3000` が公開されているため、ダミー IoT デバイスを表示する Web ページを確認できます
-   ポート `3001` は純粋にチュートリアル・アクセス用に公開されているため、cUrl または Postman は同じネットワークに
    属していなくても JSON コマンドを作成できます

`tutorial` コンテナは、次のように環境変数によって駆動されます:

| キー                    | 値                                                    | 説明                                                                                                                                   |
| ----------------------- | ----------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| DEBUG                   | `tutorial:*`                                          | ログ記録に使用されるデバッグ フラグ                                                                                                    |
| WEB_APP_PORT            | `3000`                                                | ダミー・デバイス・データを表示するウェブ・アプリで使用されるポート                                                                     |
| IOTA_HTTP_HOST          | `iot-agent`                                           | JSON 用 IoT Agent のホスト名 - 以下を参照してください。                                                                               |
| IOTA_HTTP_PORT          | `7896`                                                | JSON の IoT Agent がリッスンするポート。`7896` は HTTP 経由の JSON の一般的なデフォルトです                                            |
| DUMMY_DEVICES_PORT      | `3001`                                                | ダミー IoT デバイスがコマンドを受信するために使用するポート                                                                           |
| DUMMY_DEVICES_API_KEY   | `4jggokgpepnvsb2uv4s40d59ov`                          | JSON インタラクションに使用されるランダムなセキュリティ・キー - デバイスと IoT Agent 間の相互作用の整合性を確保するために使用されます |
| DUMMY_DEVICES_TRANSPORT | `HTTP`                                                | ダミー IoT デバイスで使用されるトランスポート・プロトコル                                                                             |
| DUMMY_DEVICES_PAYLOAD   | `JSON`                                                | ダミー IoT デバイスで使用されるペイロード形式                                                                                         |
| IOTA_JSON_LD_CONTEXT    | `http://context/user-context.jsonld`                  | デバイス・データモデルの定義に使用される `@context` ファイルの場所                                                                     |

YAML ファイルで説明されている他の `tutorial` コンテナ設定値は、このチュートリアルでは使用しません。

<a name="iot-agent-for-json-configuration"></a>

## IoT Agent for JSON  の設定

[IoT Agent for JSON](https://fiware-iotagent-json.readthedocs.io/en/latest/) は、Docker コンテナ内でインスタンス化
できます。公式の Docker イメージは、`fiware/iotagent-json` とタグ付けされた
[Docker Hub](https://hub.docker.com/r/fiware/iotagent-json/) から入手できます。必要な設定は次のとおりです:

```yaml
iot-agent:
    image: quay.io/fiware/iotagent-json:latest
    hostname: iot-agent
    container_name: fiware-iot-agent
    depends_on:
        - mongo-db
    networks:
        - default
    expose:
        - "4041"
        - "7896"
    ports:
        - "4041:4041"
        - "7896:7896"
    environment:
        - IOTA_CB_HOST=orion
        - IOTA_CB_PORT=1026
        - IOTA_NORTH_PORT=4041
        - IOTA_REGISTRY_TYPE=mongodb
        - IOTA_LOG_LEVEL=DEBUG
        - IOTA_TIMESTAMP=true
        - IOTA_CB_NGSI_VERSION=ld
        - IOTA_AUTOCAST=true
        - IOTA_MONGO_HOST=mongo-db
        - IOTA_MONGO_PORT=27017
        - IOTA_MONGO_DB=iotagentjson
        - IOTA_HTTP_PORT=7896
        - IOTA_PROVIDER_URL=http://iot-agent:4041
        - IOTA_DEFAULT_RESOURCE=/iot/json
        - IOTA_JSON_LD_CONTEXT=http://context/user-context.jsonld
        - IOTA_FALLBACK_TENANT=openiot
        - IOTA_MULTI_CORE=true
```

`iot-agent` コンテナは、Orion Context Broker の存在に依存し、MongoDB データベースを使用して、デバイス URL やキーなどのデバイス
情報を保持します。コンテナは2つのポートでリッスンしています:

-   ポート `7896` は、ダミー IoT デバイスから HTTP 経由で JSON 測定値を受信するために公開されています
-   ポート `4041` は、チュートリアルアクセス用にのみ公開されています。そのため、cUrl または Postman は、同じネットワークに
    属していなくてもプロビジョニング・コマンドを作成できます

`iot-agent` コンテナは、次のように環境変数によって駆動されます:

| キー                 | 値                                   | 説明                                                                                                                                               |
| -------------------- | ------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| IOTA_CB_HOST         | `orion`                              | コンテキストを更新する Context Broker のホスト名                                                                                                   |
| IOTA_CB_PORT         | `1026`                               | Context Broker がコンテキストを更新するためにリッスンするポート                                                                                    |
| IOTA_NORTH_PORT      | `4041`                               | IoT Agent の構成およびコンテキスト ブローカーからのコンテキスト更新の受信に使用されるポート                                                        |
| IOTA_REGISTRY_TYPE   | `mongodb`                            | IoT デバイス情報をメモリに保持するか、データベースに格納するか                                                                                     |
| IOTA_LOG_LEVEL       | `DEBUG`                              | IoT Agent のログ・レベル                                                                                                                           |
| IOTA_TIMESTAMP       | `true`                               | 接続されたデバイスから受信した各測定値でタイムスタンプ情報を提供するかどうか                                                                       |
| IOTA_CB_NGSI_VERSION | `LD`                                 | アクティブな属性の更新を送信するときに使用する NGSI-LD を供給するかどうか                                                                          |
| IOTA_AUTOCAST        | `true`                               | JSON number の値が文字列ではなく数値として読み取られるようにする                                                                                   |
| IOTA_MONGO_HOST      | `context-db`                         | mongoDB のホスト名 - デバイス情報を保持するために使用されます                                                                                      |
| IOTA_MONGO_PORT      | `27017`                              | mongoDB がリッスンしているポート                                                                                                                   |
| IOTA_MONGO_DB        | `iotagentul`                         | mongoDB で使用されるデータベースの名前                                                                                                             |
| IOTA_HTTP_PORT       | `7896`                               | IoT Agent が HTTP 経由で IoT デバイス トラフィックをリッスンするポート                                                                             |
| IOTA_PROVIDER_URL    | `http://iot-agent:4041`              | コマンドの登録時に Context Broker に渡される URL。Context Broker がデバイスにコマンドを発行するときにフォワーディング URL の場所として使用されます |
| IOTA_JSON_LD_CONTEXT | `http://context/user-context.jsonld` | デバイス データ モデルの定義に使用される `@context` ファイルの場所                                                                                 |
| IOTA_FALLBACK_TENANT | `openiot`                            | 通信から明示的なテナントを受信していない場合に使用するテナント                                                                                     |

<a name="prerequisites"></a>

# 前提条件

<a name="docker"></a>

## Docker

物事を単純にするために、両方のコンポーネントが [Docker](https://www.docker.com) を使用して実行されます。**Docker** は、
さまざまコンポーネントをそれぞれの環境に分離することを可能にするコンテナ・テクノロジです。

-   Docker Windows にインストールするには、[こちら](https://docs.docker.com/docker-for-windows/)
    の手順に従ってください
-   Docker Mac にインストールするには、[こちら](https://docs.docker.com/docker-for-mac/)の手順に従ってください
-   Docker Linux にインストールするには、[こちら](https://docs.docker.com/install/)の手順に従ってください

**Docker Compose** は、マルチコンテナ Docker アプリケーションを定義して実行するためのツールです。
[YAML file](https://raw.githubusercontent.com/FIWARE/tutorials.IoT-Agent-JSON/NGSI-LD/docker-compose/orion-ld.yml)
ファイルは、アプリケーションのために必要なサービスを構成するために使用します。つまり、すべてのコンテナ・サービスは
1 つのコマンドで呼び出すことができます。Docker Compose は、デフォルトで Docker for Windows と Docker for Mac の一部と
してインストールされますが、Linux ユーザは[ここ](https://docs.docker.com/compose/install/)に記載されている手順に
従う必要があります。

次のコマンドを使用して、現在の **Docker** バージョンと **Docker Compose** バージョンを確認できます:

```console
docker-compose -v
docker version
```

Docker バージョン 24.0.x 以降と Docker Compose 2.24.x 以上を使用していることを確認し、必要に応じてアップグレード
してください。

## WSL

シンプルな bash スクリプトを使用してサービスを開始します。Windows ユーザは [を使用して Windows に Linux をインストールする方法](https://learn.microsoft.com/ja-jp/windows/wsl/install)
をダウンロードして、Windows 上の Linux ディストリビューションと同様のコマンドライン機能を提供する必要があります。

# 起動

開始する前に、必要な Docker イメージをローカルで取得または構築しておく必要があります。リポジトリを複製し、以下の
コマンドを実行して必要なイメージを作成してください:

```console
git clone https://github.com/FIWARE/tutorials.IoT-Agent-JSON.git
cd tutorials.IoT-Agent-JSON
git checkout NGSI-LD

./services create
```

その後、リポジトリ内で提供される [services](https://github.com/FIWARE/tutorials.IoT-Agent-JSON/blob/NGSI-LD/services) Bash
スクリプトを実行することにより、コマンドラインからすべてのサービスを初期化できます:

```console
git clone https://github.com/FIWARE/tutorials.IoT-Agent-JSON.git
cd tutorials.IoT-Agent-JSON
git checkout NGSI-LD

./services [orion|scorpio|stellio]
```

> :information_source: **注:** クリーンアップしてやり直す場合は、次のコマンドを使用します:
>
> ```console
> ./services stop
> ```

<a name="provisioning-an-iot-agent"></a>

# IoT Agent のプロビジョニング

チュートリアルを正しく実行するには、ブラウザでデバイス・モニタページを使用できることを確認し、ページをクリックしてオーディオを
有効にしてから cUrl コマンドを入力してください。デバイス・モニタは、JSON 構文を使用してダミー・デバイスの配列の現在の状態を
表示します

#### デバイス・モニタ

デバイス・モニタは次の場所にあります: `http://localhost:3000/device/monitor`

<a name="checking-the-iot-agent-service-health"></a>

## IoT Agent サービスの健全性の確認

公開されたポートに HTTP リクエストを行うことで、IoT Agent が実行されているかどうかを確認できます。

#### 1️⃣ リクエスト:

```console
curl -X GET \
  'http://localhost:4041/iot/about'
```

レスポンスは次のようになります。

```json
{
    "libVersion": "2.12.0-next",
    "port": "4041",
    "baseRoot": "/",
    "version": "1.13.0-next"
}
```

> **`Failed to connect to localhost port 4041: Connection refused` というレスポンスを受け取った場合はどうなりますか？**
>
> `Connection refused` というレスポンスを受け取った場合、このチュートリアルで想定した場所で IoT Agent を見つけることが
> できません - 各 cUrl コマンドの URL とポートを正しい IP アドレスに置き換える必要があります。すべての cUrl コマンド・
> チュートリアルでは、IoT Agent が `localhost:4041` で利用可能であることを前提としています。
>
> 次の対処を試してください :
>
> -   Docker コンテナが実行されていることを確認するには、次を試してください :
>
> ```console
> docker ps
> ```
>
> 実行中の4つのコンテナが表示されます。IoT Agent が実行されていない場合、必要に応じてコンテナーを再起動できます。
> このコマンドは、開いているポート情報も表示します。
>
> -   [`docker-machine`](https://docs.docker.com/machine/) および [Virtual Box](https://www.virtualbox.org/), Context Broker,
>     IoT Agent, ダミー・デバイスをインストールした場合、Dcoker コンテナは、別の IP アドレスから実行されている可能性が
>     あります。次のように仮想ホスト IP を取得する必要があります:
>
> ```console
> curl -X GET \
>  'http://$(docker-machine ip default):4041/version'
> ```
>
> または、コンテナ・ネットワーク内からすべての curl コマンドを実行します:
>
> ```console
> docker run --network fiware_default --rm appropriate/curl -s \
>  -X GET 'http://iot-agent:4041/iot/about'
> ```

<a name="connecting-iot-devices"></a>

## IoT デバイスの接続

IoT Agent は、IoT デバイスと Context Broker 間のミドルウェアとして機能します。したがって、一意の ID を持つコンテキスト。データ
・エンティティを作成できる必要があります。サービスがプロビジョニングされ、不明なデバイスが測定を行うと、IoT Agent は、標準の
`urn:ngsi-ld:` プレフィックス、デフォルトの `type` および提供された `<device-id>` (デバイスが認識され、既知の ID
にマッピングできる場合を除きます)。

**NGSI-LD** の場合、各デバイス・エンティティのすべての属性の定義は、提供された `@context` ファイルで利用できるようにする必要が
あります。基本の**デバイス**スマート・データモデルは、
[ここ](https://swagger.lab.fiware.org/?url=https://fiware.github.io/tutorials.NGSI-LD/swagger/device.yaml)
にあります。この発見可能性とサードパーティ・システムとの相互運用性を可能にするために、IoT Agent には、リクエストごとに Context
Broker に再送信される `@context` ファイルの場所を保持する `IOTA_JSON_LD_CONTEXT` 環境変数も事前に提供する必要があります。

提供されたすべての IoT デバイス `<device-id>` が常に一意であるという保証はないため、IoT Agent へのすべての
プロビジョニング・リクエストには2つの必須ヘッダが必要です:

-   `fiware-service` (`NGSILD-Tenant` に相当) ヘッダは、特定のサービスのエンティティを別の mongoDB データベースに
    保持できるように定義します
-   `fiware-servicepath` は、デバイスの配列を区別するために使用できます

**NGSI-LD** IoT Agent は **NGSI-v2** と下位互換性があるため、現在もプロビジョニング時に古い FIWARE ヘッダの名前を
使用していることに注意してください。

たとえば、スマート・シティのアプリケーション内では、さまざまな部門 (公園, 交通機関, ゴミ収集など) に対して異なる
`fiware-service` ヘッダが期待され、各 `fiware-servicepath` は特定の公園などを参照します。つまり、各サービスのデータと
デバイスは必要に応じて識別および分離できますが、データはサイロ化されません。たとえば、公園内の **Smart Bin** のデータを、
ごみ収集車の **GPS Unit** と組み合わせて、トラックのルートを効率的に変更できます。

**Smart Bin** と**GPS Unit** は異なるメーカーのものである可能性が高く、使用されている `<device-id>` 内に重複がないことは
保証できません。`fiware-service` および ` fiware-servicepath` ヘッダを使用すると、これが常に当てはまり、Context Broker
がコンテキスト・データの元のソースを識別できるようになります。

<a name="provisioning-a-service-group"></a>

### サービス・グループのプロビジョニング

グループ・プロビジョニングの呼び出しはデバイス接続の最初のステップです。各測定で常に認証キーを提供する必要があり、IoT
Agent は最初に Context Broker がレスポンスしている URL を認識しません。

すべての匿名デバイスに対してデフォルトのコマンドと属性を設定することも可能ですが、各デバイスを個別にプロビジョニング
するため、このチュートリアルでは、これを行いません。

この例では、デバイスの匿名グループをプロビジョニングします。IoT Agent に、一連のデバイスが `IOTA_HTTP_PORT` (IoT Agent
が **Northbound** 通信をリッスンしているポート) にメッセージを送信することを伝えます。

#### 2️⃣ リクエスト:

```console
curl -iX POST 'http://localhost:4041/iot/services' \
    -H 'fiware-service: openiot' \
    -H 'fiware-servicepath: /' \
    -H 'Content-Type: application/json' \
    --data-raw '{
    "services": [
        {
            "apikey": "4jggokgpepnvsb2uv4s40d59ov",
            "cbroker": "http://orion:1026",
            "entity_type": "Device",
            "resource": "/iot/d",
            "attributes": [
                {
                    "object_id": "bpm", "type": "Property", "name": "heartRate",
                    "metadata": { "unitCode": {"type": "Text", "value": "5K" }}
                },
                {
                    "object_id": "s", "name": "status", "type": "Property"
                },
                {
                    "object_id": "gps", "name": "location", "type": "geo:point"
                }
            ],
            "static_attributes": [
                {
                    "name": "category", "type": "Property", "value": "sensor"
                },
                {
                    "name": "supportedProtocol", "type": "Property", "value": "ul20"
                }
            ]
        }
    ]
}'
```

この例では、IoT Agent は、`/iot/d` エンドポイントを使用し、デバイスがトークン `4jggokgpepnvsb2uv4s40d59ov`
を含めることによって自身を認証することを通知されます。UltraLight IoT Agent の場合、これはデバイスが GET または POST
リクエストを以下に送信することを意味します。

```
http://iot-agent:7896/iot/d?i=<device_id>&k=4jggokgpepnvsb2uv4s40d59ov
```

[以前のチュートリアル](https://github.com/FIWARE/tutorials.IoT-Sensors/tree/NGSI-LD)
でおなじみの UltraLight 構文である必要があります。

サービス・グループのプロビジョニングを使用して、`attributes` マッピングと一般的な `static_attributes` を定義することも
できます。`attributes` マッピングは、匿名の着信デバイスの一般的なエイリアスを定義します。たとえば、キー `gps` を定義して、
`location` GeoProperty をマップできます。

IoT デバイスからの測定値をリソース URL で受信した場合、それを解釈して Context Broker に渡す必要があります。`entity_type`
属性は、リクエストを行った各デバイスのデフォルトの `type` を提供します (この場合、匿名デバイスは `Device` エンティティと
呼ばれます。さらに、Context Broker (`cbroker`) の場所が必要です。IoT Agent が受信した測定値を正しい場所に渡すことが
できます。`cbroker` はオプションの属性です。指定されていない場合、IoT Agent は設定ファイルで定義されている Context
Broker URL を使用しますが、完全を期すために、ここに含まれています。

<a name="provisioning-a-sensor"></a>

### センサのプロビジョニング

NGSI-LD [仕様](https://cim.etsi.org/NGSI-LD/official/0--1.html) では、
コンテキスト・データ・エンティティを作成するときに完全な URN が義務付けられていますが、デバイスからの着信メッセージはこの
規則を認識しません。さらに、コンテキスト・データ・エンティティの属性名は、関連付けられた `@context` ファイル内にある短い名前
と一致する必要があります。これらのマッピングは、以前のリクエストで見たようにサービス・グループ・レベルで定義することも、
各デバイスを個別にプロビジョニングすることで定義することもできます。

3種類の測定属性をプロビジョニングできます:

-   `attributes` は、デバイスからのアクティブな読み取り値のマッピングです
-   `lazy` 属性はリクエスト時にのみ送信されます。IoT Agent はデバイスに測定値を返すよう通知します
-   `static_attributes` は、名前が示すように、Context Broker に渡されるデバイスに関する静的データ
    (リレーションシップなど) です

> **注**: 個々の `id` が不要な場合、または集約データで十分な場合、
> `attributes` は個別ではなくプロビジョニング・サービス内で定義できます

#### 3️⃣ リクエスト:

```console
curl -L -X POST 'http://localhost:4041/iot/devices' \
    -H 'fiware-service: openiot' \
    -H 'fiware-servicepath: /' \
    -H 'Content-Type: application/json' \
--data-raw '{
  "devices": [
    {
      "device_id": "temperature001",
      "entity_name": "urn:ngsi-ld:Device:temperature001",
      "entity_type": "Device",
      "apikey": "4jggokgpepnvsb2uv4s40d59ov",
      "timezone": "Europe/Berlin",
      "attributes": [
        {
          "object_id": "t",
          "name": "temperature",
          "type": "Property",
          "metadata": {
            "unitCode": {
              "type": "Text",
              "value": "CEL"
            }
          }
        }
      ],
      "static_attributes": [
        {
          "name": "controlledAsset",
          "type": "Relationship",
          "value": "urn:ngsi-ld:Building:barn001"
        }
      ]
    }
  ]
}'
```

リクエストでは、デバイス `temperature001` を URN `urn:ngsi-ld:Device:temperature001` に関連付け、`t` を読み取るデバイスを
コンテキスト属性 `temperature` にマッピングしています (これは、適切なメタデータを持つ**プロパティ**として定義されます)。
`managedAsset` **Relationship** は `static_attribute` としても定義され、デバイスを **Building** `urn:ngsi-ld:Building:barn001`
内に配置します。

> 静的属性は、`q` パラメータを使用したクエリを有効にするエンティティの追加データとして役立ちます。たとえば、Smart Data Models
> [Device](https://github.com/smart-data-models/dataModel.Device/blob/master/Device/doc/spec.md) モデルは、次のようにクエリを
> 実行できるようにする `category` や `ControlledProperty` などの属性を定義します:
>
> -   _現在 `batteryLevel` が低い**アクチュエータ**はどれですか？_
>
> `/ngsi-ld/v1/entities?q=category=="actuator";batteryLevel<0.1`
>
> -   _2020年1月より前にインストールされた `fillingLevel` を測定する**デバイス**はどれですか？_
>
> `/ngsi-ld/v1/entities?q=controlledProperty=="fillingLevel";dateInstalled<"2020-01-25T00:00:00.000Z"`
>
> 明らかに、静的データは必要に応じて拡張でき、エンティティ ID がクエリに対して柔軟性がない場合は、デバイスごとに一意の `name`
> や `serialNumber` などの追加データを含めることもできます。
>
> `/ngsi-ld/v1/entities?q=serialNumber=="XS403001-002"`
>
> さらに、固定の `location` 静的属性を持つデバイスは、ジオフェンスパラメータを使用してクエリすることもできます。
>
> `/ngsi-ld/v1/entities?georel=near;maxDistance:1500&geometry=point&coords=52.5162,13.3777`

次のリクエストを行うことで、**Temperature Sensor** デバイス `temperature001` からのダミー IoT デバイス測定をシミュレート
できます。

#### 4️⃣ リクエスト:

```console
curl -L -X POST 'http://localhost:7896/iot/json?k=4jggokgpepnvsb2uv4s40d59ov&i=temperature001' \
    -H 'Content-Type: application/json' \
    --data-raw '{ "t": 3}'
```

スプリンクラー・システムがアクティブ化されたときに、以前のチュートリアル (IoT Agent が接続される前) で同様の要求が行われ、
各センサーの状態が変化し、ノースバウンド・リクエストがデバイス・モニタに記録されます。

このチュートリアルでは、IoT Agent が接続されたので、サービス・グループは、IoT Agent がリッスンするエンドポイント
(`/iot/json`) とリクエストの認証に使用される API キー (`4jggokgpepnvsb2uv4s40d59ov`) を定義しました。 これらは両方とも
リクエストから認識されるため、測定は有効です。

デバイス (`temperature001`) を具体的にプロビジョニングしているため、IoT Agent は Orion Context Broker でリクエストを
生成する前に属性をマッピングできます。

Context Broker からエンティティのデータを取得することで、測定値が記録されたことを確認できます。`fiware-service` と
`fiware-service-path` ヘッダを追加することを忘れないでください。

#### 5️⃣ リクエスト:

```console
curl -G -iX GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Device:temperature001' \
    -H 'NGSILD-Tenant: openiot' \
    -H 'Accept: application/ld+json' \
    -H 'Link: <http://context/user-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
    -d 'attrs=temperature'
```

#### レスポンス:

```json
{
    "@context": [
            "http://context/user-context.jsonld",
            "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
        ],
    "id": "urn:ngsi-ld:Device:temperature001",
    "type": "Device",
    "temperature": {
        "type": "Property",
        "value": 3,
        "unitCode": "CEL",
        "observedAt": "2020-09-14T15:23:12.263Z"
    }
}
```

レスポンスは、`id=temperature001` の **Temperature Sensor** デバイスが IoT Agent によって正常に識別され、エンティティ
`id=urn:ngsi-ld:Device:temperature001` にマッピングされたことを示しています。 この新しいエンティティは、コンテキスト・データ
内に作成されています。ダミー・デバイス測定リクエストからの `t` 属性は、コンテキスト内のより意味のある `temperature`
属性にマップされています。お気づきのとおり、属性のメタデータに `observedAt` 属性が追加されました。これは、エンティティと属性
が最後に更新された時刻を表し、IoT Agnet が起動したときに、`IOTA_TIMESTAMP` 環境変数が設定されているため、新しい各エンティティ
に自動的に追加されます。

サービス・グループをプロビジョニングすることで、IoT Agent を開いて匿名デバイスから読み取り値を受信することもできます。
たとえば、必要なすべてのデータがデバイス自体から直接利用できる場合は、個々のデバイスをプロビジョニングする必要はありません。

たとえば、`/iot/json` エンドポイントへのこのリクエストについて考えてみます:

#### 6️⃣ リクエスト:

```console
curl -iX POST 'http://localhost:7896/iot/json?k=4jggokgpepnvsb2uv4s40d59ov&i=motion003' \
-H 'Content-Type: application/json' \
--data-raw '{"c": 1}'
```

リソース・エンドポイントは以前にサービス・グループ内で定義されており、API キーが一致するため、これは有効な測定値として認識
され、サービス・グループの知識に基づいてマッピングされた属性を持つ新しいエンティティが Context Broker に作成されます。

#### 7️⃣ リクエスト:

```console
curl -L -X GET 'http://localhost:1026/ngsi-ld/v1/entities/?type=Device' \
    -H 'NGSILD-Tenant: openiot' \
    -H 'Accept: application/ld+json' \
    -H 'Link: <http://context/user-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"'
```

#### レスポンス:

```json
[
    {
        "@context": [
            "http://context/user-context.jsonld",
            "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
        ],
        "id": "urn:ngsi-ld:Device:motion003",
        "type": "Device",
        "c": {
            "type": "Property",
            "value": "1",
            "observedAt": "2020-09-17T09:41:56.755Z"
        },
        "category": {
            "type": "Property",
            "value": "sensor",
            "observedAt": "2020-09-17T09:41:56.755Z"
        },
        "supportedProtocol": {
            "type": "Property",
            "value": "ul20",
            "observedAt": "2020-09-17T09:41:56.755Z"
        }
    }
]
```

ご覧のとおり、サービス・グループのエンティティ・タイプと `static_attributes` は、Context Broker のエンティティにコピーされて
いますが、測定値 `{"c":1}` にはマッピングがないため、 プロパティは、受け取った測定値から直接コピーされています。

<a name="provisioning-an-actuator"></a>

### アクチュエータのプロビジョニング

アクチュエータのプロビジョニングは、センサのプロビジョニングに似ています。今回は、`endpoint` 属性は IoT Agent が UltraLight
コマンドを送信する必要がある場所を保持し、`commands` 配列には呼び出すことができる各コマンドのリストが含まれます。
以下の例では、`deviceId=water001` で water をプロビジョニングします。エンドポイントは `http://iot-sensors:3001/iot/water001`
であり、`on` と `off` コマンドを受け入れることができます。`transport=HTTP` 属性は、使用される通信プロトコルを定義します。

#### 8️⃣ リクエスト:

```console
curl -L -X POST 'http://localhost:4041/iot/devices' \
    -H 'fiware-service: openiot' \
    -H 'fiware-servicepath: /' \
    -H 'Content-Type: application/json' \
--data-raw '{
  "devices": [
    {
      "device_id": "water001",
      "entity_name": "urn:ngsi-ld:Device:water001",
      "entity_type": "Device",
      "apikey": "4jggokgpepnvsb2uv4s40d59ov",
      "protocol": "PDI-IoTA-UltraLight",
      "transport": "HTTP",
      "endpoint": "http://iot-sensors:3001/iot/water001",
      "commands": [
        {
          "name": "on",
          "type": "Property"
        },
        {
          "name": "off",
          "type": "Property"
        }
       ],
       "static_attributes": [
         {"name":"controlledAsset", "type": "Relationship","value": "urn:ngsi-ld:Building:barn001"}
        ]
    }
  ]
}
```

Context Broker を接続する前に、`/ngsi-ld/v1/entities/` エンドポイントを使用して IoT Agent のノースポートに直接 PATCH
リクエストを行うことで、コマンドをデバイスに送信できることをテストできます。このエンドポイントは、接続すると Context
Broker によって最終的に呼び出されます。設定をテストするには、次のようにコマンドを直接実行できます:

#### 9️⃣ リクエスト:

```console
curl -L -X PATCH 'http://localhost:4041/ngsi-ld/v1/entities/urn:ngsi-ld:Device:water001/attrs/on' \
    -H 'fiware-service: openiot' \
    -H 'fiware-servicepath: /' \
    -H 'Content-Type: application/json' \
--data-raw '{

        "type": "Property",
        "value": " "

}'
```

デバイス・モニタのページを表示している場合は、water sprinkler の変化の状態も確認できます。

![](https://fiware.github.io/tutorials.IoT-Agent-JSON/img/water-on.png)

Irrigation system をオンにするコマンドの結果は、Context Broker 内のエンティティをクエリすることで読み取ることが
できます。

#### 1️⃣0️⃣ リクエスト:

```console
curl -L -X GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Device:water001' \
    -H 'NGSILD-Tenant: openiot' \
    -H 'Link: <http://context/user-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
    -H 'Accept: application/json'
```

#### レスポンス:

```json
{
    "id": "urn:ngsi-ld:Device:water001",
    "type": "Device",
    "on_status": {
        "type": "Property",
        "value": {
            "@type": "commandStatus",
            "@value": "OK"
        },
        "observedAt": "2020-09-14T15:27:11.066Z"
    },
    "on_info": {
        "type": "Property",
        "value": {
            "@type": "commandResult",
            "@value": "OK"
        },
        "observedAt": "2020-09-14T15:27:11.066Z"
    },
    "controlledAsset": {
        "type": "Relationship",
        "object": "urn:ngsi-ld:Building:barn001",
        "observedAt": "2020-09-14T15:27:11.066Z"
    }
}
```

`observedAt` は、エンティティに関連付けられたコマンドが呼び出された最後の時間を示します。`on` コマンドの結果は、
`on_info` 属性の値で見ることができます。

<a name="provisioning-a-filling-station"></a>

### Filling Station のプロビジョニング

コマンドと測定の両方を提供するデバイスのプロビジョニングは、リクエストのボディに `attributes` 属性と `command` 属性の両方を
含む HTTP POST リクエストを作成するだけです。

#### 1️⃣1️⃣ リクエスト:

```console
curl -L -X POST 'http://localhost:4041/iot/devices' \
-H 'fiware-service: openiot' \
-H 'fiware-servicepath: /' \
-H 'Content-Type: application/json' \
--data-raw '{
  "devices": [
    {
      "device_id": "filling001",
      "entity_name": "urn:ngsi-ld:Device:filling001",
      "entity_type": "FillingLevelSensor",
      "apikey": "4jggokgpepnvsb2uv4s40d59ov",
      "protocol": "PDI-IoTA-UltraLight",
      "transport": "HTTP",
      "endpoint": "http://iot-sensors:3001/iot/filling001",
      "commands": [
        {
          "name": "add",
          "type": "Property"
        },
        {
          "name": "remove",
          "type": "Property"
        }
      ],
      "attributes": [
        {
          "object_id": "f",
          "name": "fillingLevel",
          "type": "Number",
          "metadata": {
            "unitCode": {
              "type": "Text",
              "value": "C62"
            }
          }
        }
      ],
       "static_attributes": [
        {
          "name": "controlledAsset",
          "type": "Relationship",
          "value": "urn:ngsi-ld:Building:barn001"
        }
      ]
    }
  ]
}'
```

<a name="provisioning-a-tractor-fmis-system"></a>

### Tractor FMIS System のプロビジョニング

同様に、2つのコマンド (`start` と `stop`) と2つの属性を持つ **Tractor** は、次のようにプロビジョニングできます。

#### 1️⃣2️⃣ リクエスト:

```console
curl -L -X POST 'http://localhost:4041/iot/devices' \
    -H 'fiware-service: openiot' \
    -H 'fiware-servicepath: /' \
    -H 'Content-Type: application/json' \
--data-raw '{
  "devices": [
    {
      "device_id": "tractor001",
      "entity_name": "urn:ngsi-ld:Device:tractor001",
      "entity_type": "Tractor",
      "apikey": "4jggokgpepnvsb2uv4s40d59ov",
      "protocol": "PDI-IoTA-UltraLight",
      "transport": "HTTP",
      "endpoint": "http://iot-sensors:3001/iot/tractor001",
      "commands": [
        {"name": "start","type": "Property"},
        {"name": "stop","type": "Property"}
       ],
       "static_attributes": [
         {"name":"controlledAsset", "type": "Relationship","value": "urn:ngsi-ld:Building:barn001"}
        ]
    }
  ]
}
'
```

Tractor からの測定値 (例: `gps`) は、サービス・グループ内ですでに定義されているため、ここで繰り返す必要はありません。

プロビジョニングされたデバイスの完全なリストは、`/iot/devices` エンドポイントに GET リクエストを行うことで取得できます。

#### 1️⃣3️⃣ リクエスト:

```console
curl -L -X GET 'http://localhost:4041/iot/devices' \
    -H 'fiware-service: openiot' \
    -H 'fiware-servicepath: /'
```

<a name="enabling-context-broker-commands"></a>

## Context Broker コマンドの有効化

IoT Agent を IoT デバイスに接続すると、Orion Context Broker にコマンドが利用可能になったことが通知されます。
言い換えると、IoT Agent は、コマンド属性の
[コンテキスト・プロバイダ](https://github.com/FIWARE/tutorials.Context-Providers/) として登録されました。

コマンドが登録されると、Orion Context Broker にリクエストを送信することで、
[以前のチュートリアル](https://github.com/FIWARE/tutorials.IoT-Sensors) で行ったように、JSONリクエストをIoTデバイスに直接送信
するのではなく、**water** をオンにし、**Fillling System** の状態を変更し、**Irrigation System** のオンとオフを切り替えること
ができます。

<a name="activating-the-irrigation-system"></a>

### Irrigation System の有効化

`on` コマンドを呼び出すには、コンテキストで `on` 属性を更新する必要があります。

#### 1️⃣4️⃣ リクエスト:

```console
curl -L -X PATCH 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Device:water001/attrs/on' \
-H 'NGSILD-Tenant: openiot' \
-H 'Content-Type: application/json' \
-H 'Link: <http://context/user-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
--data-raw '{

        "type": "Property",
        "value": " "

}'
```

デバイス・モニタのページを表示している場合は、Water の変化の状態も確認できます。

![](https://fiware.github.io/tutorials.IoT-Agent-JSON/img/water-on.png)

<a name="activating-the-tractor"></a>

### Tractor の有効化

`start` コマンドを呼び出すには、コンテキストで `start` 属性を更新する必要があります。

#### 1️⃣5️⃣ リクエスト:

```console
curl -L -X PATCH 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Device:tractor001/attrs/start' \
    -H 'NGSILD-Tenant: openiot' \
    -H 'Content-Type: application/json' \
    -H 'Link: <http://context/user-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
--data-raw '{

        "type": "Property",
        "value": " "

}'
```

<a name="activating-the-filling-station"></a>

### Filling Station の有効化

**Fillling System** の状態を変更するには、コンテキストで `add` 属性を更新する必要があります。

#### 1️⃣6️⃣ リクエスト:

```console
curl -L -X PATCH 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Device:filling001/attrs/add' \
    -H 'NGSILD-Tenant: openiot' \
    -H 'Content-Type: application/json' \
    -H 'Link: <http://context/user-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
--data-raw '{

        "type": "Property",
        "value": " "

}'
```

<a name="service-group-crud-actions"></a>

# サービス・グループ CRUD アクション

サービス・グループをプロビジョニングするための **CRUD** 操作は、`/iot/services` エンドポイントの下で予想される HTTP
動詞にマッピングされます。

-   **Create** - HTTP POST
-   **Read** - HTTP GET
-   **Update** - HTTP PUT
-   **Delete** - HTTP DELETE

`resource` および `apikey` パラメータを使用して、サービス・グループを一意に識別します。

<a name="creating-a-service-group"></a>

### サービス・グループの作成

この例では、デバイスの匿名グループをプロビジョニングします。一連のデバイスが `IOTA_HTTP_PORT` (IoT Agent が **ノースバウンド**
通信をリッスンしている場所) にメッセージを送信することを IoT Agent に通知します。

#### 1️⃣7️⃣ リクエスト:

```console
curl -iX POST \
  'http://localhost:4041/iot/services' \
  -H 'Content-Type: application/json' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /' \
  -d '{
 "services": [
   {
     "apikey":      "4jggokgpepnvsb2uv4s40d59ov",
     "cbroker":     "http://orion:1026",
     "entity_type": "Thing",
     "resource":    "/iot/d"
   }
 ]
}'
```

<a name="read-service-group-details"></a>

### サービス・グループの詳細の読み取り

この例では、指定された `resource` パスを使用してプロビジョニングされたサービスの完全な詳細を取得します。

サービス・グループの詳細は、`/iot/services` エンドポイントに対して GET リクエストを行い、`resource` パラメータを指定すること
で読み取ることができます。

#### 1️⃣8️⃣ リクエスト:

```console
curl -X GET \
  'http://localhost:4041/iot/services?resource=/iot/d' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /'
```

#### レスポンス:

```json
{
    "count": 1,
    "services": [
        {
            "commands": [],
            "lazy": [],
            "attributes": [
                {
                    "object_id": "bpm",
                    "type": "Property",
                    "name": "heartRate",
                    "metadata": {
                        "unitCode": {
                            "type": "Text",
                            "value": "5K"
                        }
                    }
                },
                {
                    "object_id": "s",
                    "name": "status",
                    "type": "Property"
                },
                {
                    "object_id": "gps",
                    "name": "location",
                    "type": "geo:point"
                }
            ],
            "_id": "5f5f8ad8eed02a000687dec5",
            "resource": "/iot/d",
            "apikey": "4jggokgpepnvsb2uv4s40d59ov",
            "service": "openiot",
            "subservice": "/",
            "__v": 0,
            "static_attributes": [],
            "internal_attributes": [],
            "entity_type": "Device"
        }
    ]
}
```

レスポンスには、`entity_type` やデフォルトのコマンドや属性マッピングなど、各サービス・グループに関連付けられているすべての
デフォルトが含まれます。

<a name="list-all-service-groups"></a>

### すべてのサービス・グループのリスト

この例では、`/iot/services` エンドポイントに対して GET リクエストを行うことにより、プロビジョニングされたすべてのサービスを
一覧表示します。

#### 1️⃣9️⃣ リクエスト:

```console
curl -X GET \
  'http://localhost:4041/iot/services' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /'
```

#### レスポンス:

```json
{
    "count": 1,
    "services": [
        {
            "commands": [],
            "lazy": [],
            "attributes": [
                {
                    "object_id": "bpm",
                    "type": "Property",
                    "name": "heartRate",
                    "metadata": {
                        "unitCode": {
                            "type": "Text",
                            "value": "5K"
                        }
                    }
                },
                {
                    "object_id": "s",
                    "name": "status",
                    "type": "Property"
                },
                {
                    "object_id": "gps",
                    "name": "location",
                    "type": "geo:point"
                }
            ],
            "_id": "5f5f8ad8eed02a000687dec5",
            "resource": "/iot/d",
            "apikey": "4jggokgpepnvsb2uv4s40d59ov",
            "service": "openiot",
            "subservice": "/",
            "__v": 0,
            "static_attributes": [],
            "internal_attributes": [],
            "entity_type": "Device"
        }
    ]
}
```

レスポンスには、`entity_type` やデフォルトのコマンドや属性マッピングなど、各サービス・グループに関連付けられているすべての
デフォルトが含まれます。

<a name="update-a-service-group"></a>

### サービス・グループの更新

この例では、指定された `resource` パスと `apikey` で既存のサービス・グループを更新します。

サービス・グループの詳細は、`/iot/services` エンドポイントに PUT リクエストを行い、`resource` および `apikey` パラメータを
提供することで更新できます。

#### 2️⃣0️⃣ リクエスト:

```console
curl -iX PUT \
  'http://localhost:4041/iot/services?resource=/iot/d&apikey=4jggokgpepnvsb2uv4s40d59ov' \
  -H 'Content-Type: application/json' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /' \
  -d '{
  "attributes": []
}'
```

<a name="delete-a-service-group"></a>

### サービス・グループの削除

この例では、`/iot/services/` エンドポイントに DELETE リクエストを送信して、プロビジョニングされたサービス・グループを削除
します。

つまり、`http://iot-agent:7896/iot/json?i=<device_id>&k=4jggokgpepnvsb2uv4s40d59ov` (IoT Agent が**ノースバウンド**通信を
リッスンしている場所) へのリクエストが、IoT Agent によって処理されないことを意味します。削除するサービス・グループを識別する
には、`apiKey` パラメータと `resource` パラメータを指定する必要があります。

#### 2️⃣1️⃣ リクエスト:

```console
curl -iX DELETE \
  'http://localhost:4041/iot/services/?resource=/iot/d&apikey=4jggokgpepnvsb2uv4s40d59ov' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /'
```

<a name="device-crud-actions"></a>

# デバイス CRUD アクション

個々のデバイスをプロビジョニングするための **CRUD** 操作は、`/iot/devices` エンドポイントの下で予想される HTTP 動詞に
マッピングされます。

-   **Create** - HTTP POST
-   **Read** - HTTP GET
-   **Update** - HTTP PUT
-   **Delete** - HTTP DELETE

`<device-id>` を使用して、デバイスを一意に識別します。

<a name="creating-a-provisioned-device"></a>

### プロビジョニングされたデバイスの作成

この例では、個々のデバイスをプロビジョニングします。これは、`device_id=water002` をエンティティ URN `urn:ngsi-ld:water:002`
にマップし、エンティティにタイプ `water` を与えます。IoT Agent は、デバイスが2つのコマンド (`on` と `off`) を提供し、HTTP
を使用して `http://iot-sensors:3001/iot/water002` をリッスンしていることを通知されました。`attributes`, `lazy` 属性および
`static_attributes` もプロビジョニングできます。

#### 2️⃣2️⃣ リクエスト:

```console
curl -iX POST 'http://localhost:4041/iot/devices' \
    -H 'fiware-service: openiot' \
    -H 'fiware-servicepath: /' \
    -H 'Content-Type: application/json' \
--data-raw '{
  "devices": [
    {
      "device_id": "water002",
      "entity_name": "urn:ngsi-ld:Device:water002",
      "entity_type": "Device",
      "apikey": "4jggokgpepnvsb2uv4s40d59ov",
      "protocol": "PDI-IoTA-UltraLight",
      "transport": "HTTP",
      "endpoint": "http://iot-sensors:3001/iot/water002",
      "commands": [
        {
          "name": "on",
          "type": "Property"
        },
        {
          "name": "off",
          "type": "Property"
        }
       ],
       "static_attributes": [
         {"name":"controlledAsset", "type": "Relationship","value": "urn:ngsi-ld:Building:barn002"}
        ]
    }
  ]
}
'
```

<a name="read-provisioned-device-details"></a>

### プロビジョニングされたデバイスの詳細の読み取り

この例では、指定された `<device-id>` パスを使用してプロビジョニングされたデバイスの完全な詳細を取得します。

プロビジョニングされたデバイスの詳細は、`/iot/devices/<device-id>` エンドポイントに対して GET リクエストを行うことで
読み取ることができます。

#### 2️⃣3️⃣ リクエスト:

```console
curl -X GET \
  'http://localhost:4041/iot/devices/water002' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /'
```

#### レスポンス:

レスポンスには、デバイスに関連付けられているすべてのコマンドと属性のマッピングが含まれます。

```json
{
    "device_id": "water002",
    "service": "openiot",
    "service_path": "/",
    "entity_name": "urn:ngsi-ld:Device:water002",
    "entity_type": "Device",
    "apikey": "4jggokgpepnvsb2uv4s40d59ov",
    "endpoint": "http://iot-sensors:3001/iot/water002",
    "transport": "HTTP",
    "attributes": [],
    "lazy": [],
    "commands": [
        {
            "object_id": "on",
            "name": "on",
            "type": "Property"
        },
        {
            "object_id": "off",
            "name": "off",
            "type": "Property"
        }
    ],
    "static_attributes": [
        {
            "name": "controlledAsset",
            "type": "Relationship",
            "value": "urn:ngsi-ld:Building:barn002"
        }
    ],
    "protocol": "PDI-IoTA-UltraLight"
}
```

<a name="list-all-provisioned-devices"></a>

### プロビジョニングされたすべてのデバイスのリスト

この例では、`/iot/devices` エンドポイントに対して GET リクエストを行うことにより、プロビジョニングされたすべてのデバイスを
一覧表示します。

#### 2️⃣4️⃣ リクエスト:

```console
curl -X GET \
  'http://localhost:4041/iot/devices' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /'
```

#### レスポンス:

レスポンスには、すべてのデバイスに関連付けられているすべてのコマンドと属性のマッピングが含まれます。

```json
{
    "count": 5,
    "devices": [
      {
          "device_id": "water002",
          "service": "openiot",
          "service_path": "/",
          "entity_name": "urn:ngsi",
          "entity_type": "Device",
          "apikey": "4jggokgpepnvsb2uv4s40d59ov",
          "endpoint": "http://iot-sensors:3001/iot/water002",
          "transport": "HTTP",
          "attributes": [],
          "lazy": [],
          "commands": [
              {
                  "object_id": "ring",
                  "name": "ring",
                  "type": "Property"
              }
          ],
          "static_attributes": [
              {
                  "name": "controlledAsset",
                  "type": "Relationship",
                  "value": "urn:ngsi-ld:Store:002"
              }
          ],
          "protocol": "PDI-IoTA-UltraLight"
      },
      etc...
    ]
}
```

<a name="update-a-provisioned-device"></a>

### プロビジョニングされたデバイスの更新

この例では、`/iot/devices/<device-id>` エンドポイントに PUT リクエストを送信することにより、既存のプロビジョニングされた
デバイスを更新します。

#### 2️⃣5️⃣ リクエスト:

```console
curl -iX PUT \
  'http://localhost:4041/iot/devices/water002' \
  -H 'Content-Type: application/json' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /' \
  -d '{
  "entity_type": "IoT-Device"
}'
```

<a name="delete-a-provisioned-device"></a>

### プロビジョニングされたデバイスの削除

この例では、`/iot/devices/<device-id>` エンドポイントに DELETE リクエストを送信して、プロビジョニングされたデバイスを削除
します。

デバイス属性はマップされなくなり、コマンドをデバイスに送信できなくなります。デバイスがアクティブな測定を行っている場合でも、
関連するサービスが削除されていなければ、デフォルト値で処理されます。

#### 2️⃣6️⃣ リクエスト:

```console
curl -iX DELETE \
  'http://localhost:4041/iot/devices/water002' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /'
```

<a name="next-steps"></a>

# 次のステップ

高度な機能を追加することで、アプリケーションに複雑さを加える方法を知りたいですか？ このシリーズの
[他のチュートリアル](https://www.letsfiware.jp/ngsi-ld-tutorials)を読むことで見つけることができます

---

## ライセンス

[MIT](LICENSE) © 2021-2025 FIWARE Foundation e.V.
