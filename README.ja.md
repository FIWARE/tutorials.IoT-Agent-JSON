[![FIWARE Banner](https://fiware.github.io/tutorials.IoT-Agent/img/fiware.png)](https://www.fiware.org/developers)

[![FIWARE IoT Agents](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/iot-agents.svg)](https://github.com/FIWARE/catalogue/blob/master/iot-agents/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Iot-Agent.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://nexus.lab.fiware.org/repository/raw/public/badges/stackoverflow/fiware.svg)](https://stackoverflow.com/questions/tagged/fiware)
[![NGSI v2](https://img.shields.io/badge/NGSI-v2-5dc0cf.svg)](https://fiware-ges.github.io/orion/api/v2/stable/)
[![JSON](https://img.shields.io/badge/Payload-JSON-f06f38.svg)](https://fiware-iotagent-json.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
<br/> [![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)

このチュートリアルでは、
[JSON 用の IoT Agent](https://fiware-iotagent-json.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
を使用して、ダミーの [JSON](https://json.org/) ベースの IoT デバイスを接続し、
[Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/)
に送信された [NGSI-v2](https://fiware.github.io/specifications/OpenAPI/ngsiv2) リクエストを使用して、
測定値を読み取ったり、コマンドを送信したりできるようにします。

チュートリアルでは [cUrl](https://ec.haxx.se/) コマンドを使用しますが、
[Postman documentation](https://fiware.github.io/tutorials.IoT-Agent-JSON/)
としても利用できます。

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/c624b462f449c58d182b)

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
    -   [Docker と Docker Compose](#docker-and-docker-compose)
    -   [Cygwin for Windows](#cygwin-for-windows)
-   [起動](#start-up)
-   [IoT Agent のプロビジョニング](#provisioning-an-iot-agent)
    -   [IoT Agent サービスの健全性の確認](#checking-the-iot-agent-service-health)
    -   [IoT デバイスの接続](#connecting-iot-devices)
        -   [サービス・グループのプロビジョニング](#provisioning-a-service-group)
        -   [センサのプロビジョニング](#provisioning-a-sensor)
        -   [アクチュエータのプロビジョニング](#provisioning-an-actuator)
        -   [Smart Door のプロビジョニング](#provisioning-a-smart-door)
        -   [Smart Lamp のプロビジョニング](#provisioning-a-smart-lamp)
    -   [Context Broker コマンドの有効化](#enabling-context-broker-commands)
        -   [ベルを鳴らす](#ringing-the-bell)
        -   [Smart Door を開く](#opening-the-smart-door)
        -   [Smart Lamp のスイッチをオン](#switching-on-the-smart-lamp)
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
-   [次のステップ](#next-steps)

</details>

<a name="what-is-an-iot-agent"></a>

# なぜ複数の IoT Agent が必要なのですか？

> "Ils en conclurent que la syntaxe est une fantaisie et la grammaire une illusion."
>
> — Gustave Flaubert (Bouvard and Pecuchet)

IoT Agent は、デバイスのグループが独自のネイティブ・プロトコルを使用してデータ
を Context Broker に送信し、そこから管理できるようにするコンポーネントです。
すべての IoT Agent は単一のペイロード形式に対して定義されていますが、
そのペイロードに対して複数の異なるトランスポートを使用できます。。

Ultralight IoT Agent にすでに紹介しました。これは、キーと値のペアの単純なバー
(`|`) で区切られたリストを使用して通信します。このペイロードは単純で
簡潔ですが、比較的あいまいな通信メカニズムです。インターネットで使用される最も
一般的なメッセージング・ペイロードは、JavaScript 開発者に馴染みのある、いわゆる
JavaScript Object Notation または JSON です。

JSON は Ultralight よりも若干冗長ですが、より大きなメッセージを送信するコスト
は、構文の親しみやすさによって相殺されます。多くの一般的なデバイスは JSON 形式で
メッセージを送信するようにプログラムでき、データを解析するためのソフトウェア・
ライブラリが多数存在するため、個別の
[IoT Agent for JSON](https://fiware-iotagent-json.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
はこの形式で送信されるメッセージに対処するために特別に作成されました。

JSON ペイロードを使用した通信と Ultralight プレーン・テキスト・ペイロードを
使用した通信の間に実際的な違いはありません。その通信の基礎、言い換えれば、
コンポーネント間でメッセージが渡される方法を定義する基本的なプロトコルは
同じままです。明らかに、IoT Agent 内の JSON ペイロードの解析、つまり、JSON から
NGSI へのメッセージの変換、またはその逆変換は、JSON IoT Agent に固有です。

2つの IoT Agent の直接比較を以下に示します:

| IoT Agent for Ultralight                                                   | IoT Agent for JSON                                                         | 関連するプロトコル領域     |
| -------------------------------------------------------------------------- | -------------------------------------------------------------------------- | -------------------------- |
| サンプル測定値 `c\|1`                                                      | サンプル測定値 `{"count": "1"}`                                            | メッセージ・ペイロード     |
| サンプル・コマンド `Robot1@turn\|left`                                     | サンプル・コマンド `{"Robot1": {"turn": "left"}}`                          | メッセージ・ペイロード     |
| コンテンツ・タイプは `text/plain`                                          | コンテンツ・タイプは `application/json`                                    | メッセージ・ペイロード     |
| 3つのトランスポートを提供 - HTTP, MQTT, AMPQ                               | 3つのトランスポートを提供 - HTTP, MQTT, AMPQ                               | トランスポート・メカニズム |
| HTTP はデフォルトで `iot/d` の測定をリッスン                               | HTTP はデフォルトで `iot/json` の測定をリッスン                            | トランスポート・メカニズム |
| HTTP デバイスはパラメータ `?i=XXX&k=YYY` で識別                            | HTTP デバイスはパラメータ `?i=XXX&k=YYY` で識別                            | デバイスの識別             |
| 既知の URL にポストされた HTTP コマンド - レスポンスはリプライに含まれます | 既知の URL にポストされた HTTP コマンド - レスポンスはリプライに含まれます | 通信ハンドシェイク         |
| MQTT デバイスは、トピック `/XXX/YYY` のパスで識別されます                  | MQTT デバイスは、トピック  `/XXX/YYY` のパスで識別されます                 | デバイスの識別             |
| `cmd` トピックに投稿された MQTT コマンド                                   | `cmd` トピックに投稿された MQTT コマンド                                   | 通信ハンドシェイク         |
| `cmdexe` トピックに投稿された MQTT コマンド・レスポンス                    | `cmdexe` トピックに投稿された MQTT コマンド                                | 通信ハンドシェイク         |

ご覧のように、メッセージ・ペイロードは2つの IoT Agent 間で完全に異なりますが、
プロトコルの残りの多くは同じままです。

<a name="southbound-traffic-commands"></a>

## サウスバウンド・トラフィック (コマンド)

Orion Context Broker によって生成され、IoT Agent 経由で、IoT デバイスに対して
渡される HTTP リクエストは、サウスバウンド・トラフィックと呼ばれます。
サウスバウンド・トラフィックは、アクションによって実世界の状態を変更する
アクチュエータ・デバイスに対して行われる**コマンド**で構成されます。

たとえば、実際の JSON **Smart Lamp** を有効にするには、次の相互作用が発生します。

1. NGSI PATCH リクエストが **Context broker** に送信され、**Smart Lamp** の
   現在のコンテキストが更新されます

-   これは事実上、**Smart Lamp** の `on` コマンドを呼び出す間接的なリクエストです

2. **Context broker** は、コンテキスト内のエンティティを見つけ、この属性の
   コンテキスト・プロビジョニングが IoT Agent に委任されたことを確認します
3. **Context broker** は NGSI リクエストを **IoT Agent** のノース・ポートに送信して
   コマンドを呼び出します
4. **IoT Agent** はこのサウスバウンド・リクエストを受信し、JSON 構文に変換して
   **Smart Lamp** に渡します
5. **Smart Lamp** はランプをオンにし、コマンドの結果を JSON 構文で **IoT Agent**
   に返します
6. **IoT Agent** は、このノースバウンド・リクエストを受信し、解釈し、
   **Context broker** に NGSI リクエストを送信することにより、相互作用の結果を
   コンテキストに渡します
7. **Context broker** はこのノースバウンド・リクエストを受信し、コマンドの結果で
   コンテキストを更新します

![](https://fiware.github.io/tutorials.IoT-Agent-JSON/img/command-swimlane.png)

-   **ユーザ** と **Context broker** 間のリクエストは NGSI を使用します
-   **Context broker** と**IoT Agent** 間のリクエストは NGSI を使用します
-   **IoT Agent** と**IoT デバイス** 間のリクエストはネイティブ・プロトコルを
    使用します
-   **IoT デバイス**と**IoT Agent** 間のリクエストはネイティブ・プロトコルを
    使用します
-   **IoT Agent** と **Context broker** の間のリクエストは NGSI を使用します

<a name="northbound-traffic-measurements"></a>

## ノースバウンド・トラフィック (測定値)

IoT デバイスから生成され、IoT Agent を介して、Context Broker に返送される
リクエストは、ノースバウンド・トラフィックと呼ばれます。ノースバウンド・
トラフィックは、センサ・デバイスによって行われた**測定値**で構成され、
実世界の状態をシステムのコンテキスト・データに中継します。

たとえば、実際の **Motion Sensor** がカウント測定値を送信する場合、
次の相互作用が発生します。

1.  **Motion Sensor**は測定を行い、結果を **IoT Agent** に渡します
2.  **IoT Agent** は、このノースバウンド・リクエストを受信し、結果を JSON
    構文から変換し、**Context broker** に NGSI リクエストを送信することにより、
    相互作用の結果をコンテキストに渡します
3.  **Context broker** は、このノースバウンド・リクエストを受信し、
    測定結果でコンテキストを更新します

![](https://fiware.github.io/tutorials.IoT-Agent-JSON/img/measurement-swimlane.png)

-   **IoT-Device** と**IoT-Agent** 間のリクエストはネイティブ・プロトコルを
    使用します
-   **IoT-Agent** と**Context-Broker** 間のリクエストは NGSI を使用します

> **注** より複雑な対話も可能ですが、この概要は IoT Agent の基本原則を理解する
> のに十分です

<a name="common-functionality"></a>

## 共通機能

以前のセクションからわかるように、各 IoT Agent は異なるプロトコルを
解釈するので一意ですが、IoT Agent 間にはかなりの類似性があります。

-   デバイスの更新をリッスンする標準の場所を提供する
-   コンテキスト・データの更新をリッスンする標準の場所を提供する
-   デバイスのリストを保持し、コンテキスト・データ属性をデバイス構文に
    マッピングする
-   セキュリティ認証

この基本機能は、一般的な
[IoT Agentフレームワークライブラリ](https://iotagent-node-lib.readthedocs.io/)
によって抽象化されています。

#### デバイス・モニタ

このチュートリアルの目的のために、一連のダミー IoT デバイスが作成され、
Context Broker に接続されます。使用されるアーキテクチャとプロトコルの詳細は、
[IoT Sensors チュートリアル](https://github.com/FIWARE/tutorials.IoT-Sensors)
で確認できます。各デバイスの状態は、JSON デバイス・モニタの Web ページで
確認できます : `http://localhost:3000/device/monitor`

![FIWARE Monitor](https://fiware.github.io/tutorials.IoT-Agent-JSON/img/device-monitor.png)

<a name="architecture"></a>

# アーキテクチャ

このアプリケーションは、
[以前のチュートリアル](https://github.com/FIWARE/tutorials.Subscriptions/)
で作成されたコンポーネントに基づいています。
[Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) と
[IoT Agent for JSON](https://fiware-iotagent-json.readthedocs.io/en/latest/)
の2つの FIWARE コンポーネントを使用します。Orion Context Broker の使用は、
アプリケーションが _"Powered by FIWARE"_ として認定されるのに十分です。
Orion Context Broker と IoT Agent はどちらも、オープンソースの
[MongoDB](https://www.mongodb.com/) テクノロジーに依存して、保持する情報の
永続性を維持しています。 また、
[以前のチュートリアル](https://github.com/FIWARE/tutorials.IoT-Sensors/)
で作成したダミー IoT デバイスを使用します。

したがって、全体的なアーキテクチャは次の要素で構成されます :

-   FIWARE
    [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) は
    、[NGSI-v2](https://fiware.github.io/specifications/OpenAPI/ngsiv2) を使用して
    リクエストを受信します
-   FIWARE [IoT Agent for JSON](https://fiware-iotagent-json.readthedocs.io/en/latest/)は、[NGSI-v2](https://fiware.github.io/specifications/OpenAPI/ngsiv2) を使用してサウスバウンド・リクエストを受信し、それらをデバイスの [JSON](https://fiware-iotagent-json.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual) コマンドに変換します
-   基礎となる [MongoDB](https://www.mongodb.com/) データベース:
    -   **Orion Context Broker** がデータ・エンティティ、サブスクリプション、
        レジストレーションなどのコンテキスト・データ情報を保持するために
        使用します
    -   **IoT Agent** がデバイスの URL やキーなどのデバイス情報を保持する
        ために使用します
-   **Context Provider NGSI** プロキシは、このチュートリアルでは使用しません
    が、次のことを行います :
    -   [NGSI-v2](https://fiware.github.io/specifications/OpenAPI/ngsiv2)
        を使用してリクエストを受信します
    -   独自のフォーマットの APIs を使用して、公開されているデータソースへの
        リクエストを行います
    -   コンテキスト・データを
        [NGSI-v2](https://fiware.github.io/specifications/OpenAPI/ngsiv2)
        形式でOrion Context Brokerに返します
-   **ストック管理フロントエンド**は、このチュートリアルでは使用しませんが、
        次のことを行います :
    -   店舗情報の表示します
    -   各店舗で購入できる製品を表示します
    -   ユーザが製品を "購入2 して在庫数を減らすことを許可します
-   HTTP 上で実行される
    [JSON](https://fiware-iotagent-json.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
    プロトコルを使用する、
    [ダミー IoT デバイス](https://github.com/FIWARE/tutorials.IoT-Sensors)
    のセットとして機能する Web サーバ

要素間の相互作用はすべて HTTP リクエストによって開始されるため、エンティティは
コンテナ化され、公開されたポートから実行できます。

![](https://fiware.github.io/tutorials.IoT-Agent-JSON/img/architecture.png)

IoT デバイスと IoT Agent を接続するために必要な設定情報は、関連する
`docker-compose.yml` ファイルのサービス・セクションで確認できます。

<a name="dummy-iot-devices-configuration"></a>

## ダミー IoT デバイスの設定

```yaml
tutorial:
    image: fiware/tutorials.context-provider
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
```

`tutorial` コンテナは2つのポートでリッスンしています:

-   ポート `3000` が公開されているため、ダミー IoT デバイスを表示する
    Web ページを確認できます
-   ポート `3001` は純粋にチュートリアル・アクセス用に公開されているため、
    cUrl または Postman は同じネットワークに属していなくても
    JSON コマンドを作成できます

`tutorial` コンテナは、次のように環境変数によって駆動されます :

| キー                    | 値                           | 説明                                                                                                                                         |
| ----------------------- | ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| DEBUG                   | `tutorial:*`                 | ロギングに使用されるデバッグ・フラグ                                                                                                         |
| WEB_APP_PORT            | `3000`                       | ダミー・デバイスのデータを表示するweb-app が使用するポート                                                                                   |
| IOTA_HTTP_HOST          | `iot-agent`                  | IoT Agent for JSON のホスト名 - 以下を参照                                                                                                   |
| IOTA_HTTP_PORT          | `7896`                       | IoT Agent for JSON がリッスンするポート。`7896` は JSON over HTTP の一般的なデフォルトです                                                   |
| DUMMY_DEVICES_PORT      | `3001`                       | ダミー IoT デバイスがコマンドを受信するために使用するポート                                                                                  |
| DUMMY_DEVICES_API_KEY   | `4jggokgpepnvsb2uv4s40d59ov` | IoT インタラクションに使用されるランダム・セキュリティ・キー - デバイスと IoT Agent 間のインタラクションの整合性を確保するために使用されます |
| DUMMY_DEVICES_TRANSPORT | `HTTP`                       | ダミー IoT デバイスが使用するトランスポート・プロトコル                                                                                      |
| DUMMY_DEVICES_PAYLOAD   | `JSON`                       | ダミー IoT デバイスによるメッセージ・ペイロード・プロトコル                                                                                  |

YAML ファイルで説明されている他の `tutorial` コンテナ設定値は、
このチュートリアルでは使用しません。

<a name="iot-agent-for-json-configuration"></a>

## IoT Agent for JSON  の設定

[IoT Agent for JSON](https://fiware-iotagent-json.readthedocs.io/en/latest/)
は、Docker コンテナ内でインスタンス化できます。公式の Docker イメージは、
`fiware/iotagent-json` とタグ付けされた
[Docker Hub] (https://hub.docker.com/r/fiware/iotagent-json/)
から入手できます。必要な設定は次のとおりです:

```yaml
iot-agent:
    image: fiware/iotagent-json:latest
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
        - IOTA_CB_NGSI_VERSION=v2
        - IOTA_AUTOCAST=true
        - IOTA_MONGO_HOST=mongo-db
        - IOTA_MONGO_PORT=27017
        - IOTA_MONGO_DB=iotagentjson
        - IOTA_HTTP_PORT=7896
        - IOTA_PROVIDER_URL=http://iot-agent:4041
        - IOTA_DEFAULT_RESOURCE=/iot/json
```

`iot-agent` コンテナは、Orion Context Broker の存在に依存し、MongoDB
 データベースを使用して、デバイス URL やキーなどのデバイス情報を保持します。
コンテナは2つのポートでリッスンしています:

-   ポート `7896` は、ダミー IoT デバイスから HTTP 経由で JSON 測定値を
    受信するために公開されています
-   ポート `4041` は、チュートリアルアクセス用にのみ公開されています。
    そのため、cUrl または Postman は、同じネットワークに属していなくても
    プロビジョニングコマンドを作成できます

`iot-agent` コンテナは、次のように環境変数によって駆動されます:

| キー                 | 値                      | 説明                                                                                                                                  :|
| -------------------- | ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| IOTA_CB_HOST         | `orion`                 | コンテキストを更新するContext Broker のホスト名                                                                                       |
| IOTA_CB_PORT         | `1026`                  | Context Broker がコンテキストを更新するためにリッスンするポート                                                                       |
| IOTA_NORTH_PORT      | `4041`                  | IoT Agent の設定および Context Broker からのコンテキスト更新の受信に使用されるポート                                                  |
| IOTA_REGISTRY_TYPE   | `mongodb`               | メモリまたはデータベースに IoT デバイス情報を保持するかどうか                                                                         |
| IOTA_LOG_LEVEL       | `debug`                 | IoT Agent のログ・レベル                                                                                                              |
| IOTA_TIMESTAMP       | `true`                  | 接続デバイスから受信した各測定値にタイムスタンプ情報を提供するかどうか                                                                |
| IOTA_CB_NGSI_VERSION | `v2`                    | アクティブな属性の更新を送信する際に NGSI v2 を使用するかどうか                                                                       |
| IOTA_AUTOCAST        | `true`                  | JSON の数値が文字列ではなく数値として読み取られるようにする                                                                           |
| IOTA_MONGO_HOST      | `context-db`            | mongoDB のホスト名 - デバイス情報の保持に使用                                                                                         |
| IOTA_MONGO_PORT      | `27017`                 | mongoDB がリッスンしているポート                                                                                                      |
| IOTA_MONGO_DB        | `iotagentjson`          | mongoDB で使用されるデータベースの名前                                                                                                |
| IOTA_HTTP_PORT       | `7896`                  | IoT Agent が HTTP 経由で IoT デバイスのトラフィックをリッスンするポート                                                               |
| IOTA_PROVIDER_URL    | `http://iot-agent:4041` | コマンドの登録時に Context Broker に渡される URL。ContextBroker がコマンドをデバイスに発行するときに転送 URL の場所として使用されます |
| IOTA_DEFAULT_RESOURCE| `/iot/json`             | IoT Agent が JSON 測定値のリッスンを使用するデフォルトのパス                                                                          |

<a name="prerequisites"></a>

# 前提条件

<a name="docker-and-docker-compose"></a>

## Docker と Docker Compose

物事を単純にするために、両方のコンポーネントが [Docker](https://www.docker.com)
を使用して実行されます。**Docker** は、さまざまコンポーネントをそれぞれの環境に
分離することを可能にするコンテナ・テクノロジです。

-   Docker Windows にインストールするには
    、[こちら](https://docs.docker.com/docker-for-windows/)の手順に従ってくださ
    い
-   Docker Mac にインストールするには
    、[こちら](https://docs.docker.com/docker-for-mac/)の手順に従ってください
-   Docker Linux にインストールするには
    、[こちら](https://docs.docker.com/install/)の手順に従ってください

**Docker Compose** は、マルチコンテナ Docker アプリケーションを定義して実行する
ためのツールです
。[YAML file](https://raw.githubusercontent.com/Fiware/tutorials.IoT-Agent/master/docker-compose.yml)
ファイルは、アプリケーションのために必要なサービスを構成するために使用します。つ
まり、すべてのコンテナ・サービスは 1 つのコマンドで呼び出すことができます
。Docker Compose は、デフォルトで Docker for Windows と Docker for Mac の一部と
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

<a name="cygwin-for-windows"></a>

## Cygwin for Windows

シンプルな bash スクリプトを使用してサービスを開始します。Windows ユーザは
[cygwin](http://www.cygwin.com/) をダウンロードして、Windows 上の Linux ディスト
リビューションと同様のコマンドライン機能を提供する必要があります。

# 起動

開始する前に、必要な Docker イメージをローカルで取得または構築しておく必要があり
ます。リポジトリを複製し、以下のコマンドを実行して必要なイメージを作成してくださ
い :

```console
git clone https://github.com/FIWARE/tutorials.IoT-Agent.git
cd tutorials.IoT-Agent

./services create
```

その後、リポジトリ内で提供される
、[services](https://github.com/FIWARE/tutorials.IoT-Agent/blob/master/services)
Bash スクリプトを実行することによって、コマンドラインからすべてのサービスを初期
化することができます :

```console
./services start
```

> :information_source: **注:** クリーンアップしてやり直す場合は、
> 次のコマンドを使用します:
>
> ```console
> ./services stop
> ```

<a name="provisioning-an-iot-agent"></a>

# IoT Agent のプロビジョニング

チュートリアルを正しく実行するには、ブラウザでデバイス・モニタページを使用できる
ことを確認し、ページをクリックしてオーディオを有効にしてからcUrl コマンドを入力
してください。デバイス・モニタは、JSON 構文を使用してダミー・デバイスの配列の
現在の状態を表示します

#### デバイス・モニタ

デバイス・モニタは次の場所にあります: `http:// localhost:3000 / device / monitor`

<a name="checking-the-iot-agent-service-health"></a>

## IoT Agent サービスの健全性の確認

公開されたポートに HTTP リクエストを行うことで、IoT Agent が実行されているか
どうかを確認できます。

#### :one: リクエスト:

```console
curl -X GET \
  'http://localhost:4041/iot/about'
```

レスポンスは次のようになります。

```json
{
    "libVersion": "2.6.0-next",
    "port": "4041",
    "baseRoot": "/",
    "version": "1.12.0-next"
}
```

> **`Failed to connect to localhost port 4041: Connection refused` というレスポンスを受け取った場合はどうなりますか？**
>
> `Connection refused` というレスポンスを受け取った場合、このチュートリアルで
> 想定した場所で IoT Agent を見つけることができません - 各 cUrl コマンドの
> URL とポートを正しい IP アドレスに置き換える必要があります。すべての cUrl
> コマンド・チュートリアルでは、IoT Agent が `localhost:4041` で利用可能である
> ことを前提としています。
>
> 次の対処を試してください :
>
> -   Docker コンテナが実行されていることを確認するには、次を試してください :
>
> ```console
> docker ps
> ```
>
> 実行中の4つのコンテナが表示されます。IoT Agent が実行されていない場合、
> 必要に応じてコンテナーを再起動できます。
> このコマンドは、開いているポート情報も表示します。
>
> -   [`docker-machine`](https://docs.docker.com/machine/) および
>     [Virtual Box](https://www.virtualbox.org/), Context Broker, IoT Agent,
>     ダミー・デバイスをインストールした場合、Dcoker コンテナは、別の IP
>     アドレスから実行されている可能性があります。次のように仮想ホスト IP
>     を取得する必要があります :
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

IoT Agent は、IoT デバイスと Context Broker 間のミドルウェアとして機能します。
したがって、一意の IDs を持つコンテキスト・データのエンティティを作成できる
必要があります。サービスがプロビジョニングされ、不明なデバイスが測定を行うと、
IoT Agent は提供された `<device-id>` を使用してこれをコンテキストに追加します
(デバイスが認識され、既知の ID にマッピングできる場合を除きます)。

提供されたすべての IoT デバイス `<device-id>` が常に一意であるという保証は
ないため、IoT Agent へのすべてのプロビジョニング・リクエストには2つの
必須ヘッダが必要です。

-   `fiware-service` ヘッダは、特定のサービスのエンティティを別の mongoDB
    データベースに保持できるように定義します
-   `fiware-servicepath` は、デバイスの配列を区別するために使用できます

たとえば、スマート・シティのアプリケーション内では、さまざまな部門
(公園, 交通機関, ゴミ収集など) に対して異なる `fiware-service` ヘッダが
期待され、各 `fiware-servicepath` は特定の公園などを参照します。つまり、
各サービスのデータとデバイスは必要に応じて識別および分離できますが、データは
サイロ化されません。たとえば、公園内の **Smart Bin** のデータを、ごみ収集車の
**GPS Unit** と組み合わせて、トラックのルートを効率的に変更できます。

**Smart Bin** と**GPS Unit** は異なるメーカーのものである可能性が高く、
使用されている `<device-id>` 内に重複がないことは保証できません。
`fiware-service` および ` fiware-servicepath` ヘッダを使用すると、これが常に
当てはまり、Context Broker がコンテキスト・データの元のソースを
識別できるようになります。

<a name="provisioning-a-service-group"></a>

### サービス・グループのプロビジョニング

グループ・プロビジョニングの呼び出しはデバイス接続の最初のステップです。
各測定で常に認証キーを提供する必要があり、IoT Agent は最初に Context Broker
がレスポンスしている URL を認識しません。

すべての匿名デバイスに対してデフォルトのコマンドと属性を設定することも可能
ですが、各デバイスを個別にプロビジョニングするため、このチュートリアルでは
これを行いません。

この例では、デバイスの匿名グループをプロビジョニングします。IoT Agent に、
一連のデバイスが `IOTA_HTTP_PORT` (IoT Agent が **Northbound **通信を
リッスンしているポート) にメッセージを送信することを伝えます。

#### :two: リクエスト:

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
     "resource":    "/iot/json"
   }
 ]
}'
```

この例では、IoT Agent は、`/iot/json` エンドポイントを使用し、デバイスが
トークン `4jggokgpepnvsb2uv4s40d59ov` を含めることによって自身を認証する
ことを通知されます。JSON IoT Agent の場合、これはデバイスが GET または
POST リクエストを以下に送信することを意味します。

```
http://iot-agent:7896/iot/json?i=<device_id>&k=4jggokgpepnvsb2uv4s40d59ov
```

これは、Ultralight IoT Agent と非常によく似た構文です - パスのみが
変更されています。これにより、複数の IoT Agent が異なる場所でリッスン
できます。

IoT デバイスからの測定値をリソース URL で受信した場合、それを解釈して
Context Broker に渡す必要があります。`entity_type` 属性は、リクエストを
行った各デバイスのデフォルトの `type` を提供します (この場合、匿名デバイスは
`Thing` エンティティと呼ばれます。さらに、Context Broker (`cbroker`) の場所
が必要です。IoT Agent が受信した測定値を正しい場所に渡すことができます。
`cbroker` はオプションの属性です。指定されていない場合、IoT Agent は設定
ファイルで定義されている Context Broker URL を使用しますが、完全を期すために
ここに含まれています。

<a name="provisioning-a-sensor"></a>

### センサのプロビジョニング

エンティティを作成するときは、
NGSI-LD [仕様](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.01.01_60/gs_CIM009v010101p.pdf)
に従って URN を使用することをお勧めします。さらに、データ属性を定義する
ときに意味のある名前を理解する方が簡単です。これらのマッピングは、
デバイスを個別にプロビジョニングすることで定義できます。

3種類の測定属性をプロビジョニングできます。

-   `attributes` はデバイスからのアクティブな読み取り値です
-   `lazy` 属性はリクエスト時にのみ送信されます。IoT Agent はデバイスに
    測定値を返すよう通知します
-   `static_attributes` は、名前が示すように、Context Broker に渡される
    デバイスに関する静的データ (リレーションシップなど) です

> **注**: 個々の `id` が不要な場合、または集約データで十分な場合、
> `attributes` は個別ではなくプロビジョニング・サービス内で定義できます

#### :three: リクエスト:

```console
curl -iX POST \
  'http://localhost:4041/iot/devices' \
  -H 'Content-Type: application/json' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /' \
  -d '{
 "devices": [
   {
     "device_id":   "motion001",
     "entity_name": "urn:ngsi-ld:Motion:001",
     "entity_type": "Motion",
     "timezone":    "Europe/Berlin",
     "attributes": [
       { "object_id": "c", "name": "count", "type": "Integer" }
     ],
     "static_attributes": [
       { "name":"refStore", "type": "Relationship", "value": "urn:ngsi-ld:Store:001"}
     ]
   }
 ]
}
'
```

リクエストでは、デバイス `motion001` を URN `urn:ngsi-ld:Motion:001` に
関連付け、デバイスが読み取った `c` をコンテキスト属性 `count` (`Integer`
として定義) にマッピングしています。`refStore` は `static_attribute` として
定義され、デバイスを **ストア** `urn:ngsi-ld:Store:001` 内に配置します。

次のリクエストを行うことで、**Motion Sensor** デバイス `motion001` からの
ダミー IoT デバイスの測定値をシミュレートできます。

#### :four: リクエスト:

```console
curl -iX POST \
  'http://localhost:7896/iot/json?k=4jggokgpepnvsb2uv4s40d59ov&i=motion001' \
  -H 'Content-Type: application/json' \
  -d '{"c": "1"}'
```

ペイロードと `Content-Type` の両方が更新されました。ダミー・デバイスは、
ドアのロックが解除されたときに、以前のチュートリアルで同様の Ultralight
リクエストを行いました。各モーション・センサの状態が変化し、ノースバウンド・
リクエストがデバイス・モニタに記録されます。

これで IoT Agent が接続され、サービス・グループは IoT Agent がリッスンする
リソース (`iot/json`) とリクエストの認証に使用されるAPIキー
(`4jggokgpepnvsb2uv4s40d59ov`)  を定義しました。これらの両方が認識されるため、
測定値は有効です。

デバイス (`motion001`) を具体的にプロビジョニングしているため、IoT Agent は
Orion Context Broker でリクエストを生成する前に属性をマッピングできます。

Context Broker からエンティティのデータを取得することで、測定値が記録された
ことを確認できます。`fiware-service` と `fiware-service-path` ヘッダを
追加することを忘れないでください。

#### :five: リクエスト:

```console
curl -X GET \
  'http://localhost:1026/v2/entities/urn:ngsi-ld:Motion:001?type=Motion' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /'
```

#### レスポンス:

```json
{
    "id": "urn:ngsi-ld:Motion:001",
    "type": "Motion",
    "TimeInstant": {
        "type": "ISO8601",
        "value": "2018-05-25T10:51:32.00Z",
        "metadata": {}
    },
    "count": {
        "type": "Integer",
        "value": "1",
        "metadata": {
            "TimeInstant": {
                "type": "ISO8601",
                "value": "2018-05-25T10:51:32.646Z"
            }
        }
    },
    "refStore": {
        "type": "Relationship",
        "value": "urn:ngsi-ld:Store:001",
        "metadata": {
            "TimeInstant": {
                "type": "ISO8601",
                "value": "2018-05-25T10:51:32.646Z"
            }
        }
    }
}
```

このレスポンスは、 `id=motion001` を持つ **Motion Sensor** デバイスが IoT Agent
 によって正常に識別され、エンティティ `id=urn:ngsi-ld:Motion:001` にマッピング
されたことを示しています。この新しいエンティティは、コンテキスト・データ内に
作成されています。ダミー・デバイスの測定値リクエストの `c` 属性は、コンテキスト
内のより意味のある`count` 属性にマッピングされています。お気づきのとおり、
エンティティと属性のメタデータの両方に `TimeInstant` 属性が追加されています。
これは、エンティティと属性が最後に更新された時間を表し、IoT Agentの起動時に
`IOTA_TIMESTAMP` 環境変数が設定されたため、新しい各エンティティに自動的に追加
されます。`refStore` 属性は、デバイスがプロビジョニングされたときに設定された
`static_attributes` から取得されます。

<a name="provisioning-an-actuator"></a>

### アクチュエータのプロビジョニング

アクチュエータのプロビジョニングは、センサのプロビジョニングに似ています。
今回は、`endpoint` 属性は IoT Agent が JSON コマンドを送信する必要がある場所を
保持し、`commands` 配列には呼び出すことができる各コマンドのリストが含まれます。
以下の例では、`deviceId=bell001` でベルをプロビジョニングします。エンドポイント
は `http://iot-sensors:3001/iot/bell001` であり、`ring` コマンドを受け入れる
ことができます。`transport=HTTP` 属性は、使用される通信プロトコルを定義します。

#### :six: リクエスト:

```console
curl -iX POST \
  'http://localhost:4041/iot/devices' \
  -H 'Content-Type: application/json' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /' \
  -d '{
  "devices": [
    {
      "device_id": "bell001",
      "entity_name": "urn:ngsi-ld:Bell:001",
      "entity_type": "Bell",
      "transport": "HTTP",
      "endpoint": "http://iot-sensors:3001/iot/bell001",
      "commands": [
        { "name": "ring", "type": "command" }
       ],
       "static_attributes": [
         {"name":"refStore", "type": "Relationship","value": "urn:ngsi-ld:Store:001"}
      ]
    }
  ]
}
'
```

Context Broker を接続する前に、`/v2/op/update` エンドポイントを使用して
IoT Agent のノースポートに直接 REST リクエストを行うことで、コマンドをデバイス
に送信できることをテストできます。このエンドポイントは、接続すると
Context Broker によって最終的に呼び出されます。設定をテストするには、次のよう
にコマンドを直接実行できます:

#### :seven: リクエスト:

```console
curl -iX POST \
  http://localhost:4041/v2/op/update \
  -H 'Content-Type: application/json' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /' \
  -d '{
    "actionType": "update",
    "entities": [
        {
            "type": "Bell",
            "id": "urn:ngsi-ld:Bell:001",
            "ring" : {
                "type": "command",
                "value": ""
            }
        }
    ]
}'
```

デバイス・モニタのページを表示している場合は、ベルの変化の状態も確認できます。

![](https://fiware.github.io/tutorials.IoT-Agent-JSON/img/bell-ring.gif)

ベルを鳴らすコマンドの結果は、Orion Context Broker 内のエンティティをクエリ
することで読み取ることができます。

#### :eight: リクエスト:

```console
curl -X GET \
  'http://localhost:1026/v2/entities/urn:ngsi-ld:Bell:001?type=Bell&options=keyValues' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /'
```

#### レスポンス:

```json
{
    "id": "urn:ngsi-ld:Bell:001",
    "type": "Bell",
    "TimeInstant": "2018-05-25T20:06:28.00Z",
    "refStore": "urn:ngsi-ld:Store:001",
    "ring_info": " ring OK",
    "ring_status": "OK",
    "ring": ""
}
```

`TimeInstant` は、エンティティに関連付けられたコマンドが呼び出された最後の
時間を示します。`ring` コマンドの結果は、`ring_info` 属性の値で見ることが
できます。

<a name="provisioning-a-smart-door"></a>

### Smart Door のプロビジョニング

基盤となる Ultralight プロトコルと JSON プロトコルは非常に類似しているため、
アクチュエータとデバイスは、IoT Agent がデバイスと通信するために必要なデータと
同じ属性を使用してプロビジョニングされ、NGSI を JSON に解析するペイロードは
IoT Agent 自体に委任されます。コマンドと測定値の両方を提供するデバイスの
プロビジョニングは、リクエストの本文に `attributes` と `command` の両方の属性
を指定して HTTP POST リクエストを行うだけです。

#### :nine: リクエスト:

```console
curl -iX POST \
  'http://localhost:4041/iot/devices' \
  -H 'Content-Type: application/json' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /' \
  -d '{
  "devices": [
    {
      "device_id": "door001",
      "entity_name": "urn:ngsi-ld:Door:001",
      "entity_type": "Door",
      "transport": "HTTP",
      "endpoint": "http://iot-sensors:3001/iot/door001",
      "commands": [
        {"name": "unlock","type": "command"},
        {"name": "open","type": "command"},
        {"name": "close","type": "command"},
        {"name": "lock","type": "command"}
       ],
       "attributes": [
        {"object_id": "s", "name": "state", "type":"Text"}
       ],
       "static_attributes": [
         {"name":"refStore", "type": "Relationship","value": "urn:ngsi-ld:Store:001"}
       ]
    }
  ]
}
'
```

<a name="provisioning-a-smart-lamp"></a>

### Smart Lamp のプロビジョニング

同様に、2つのコマンド (`on` と `off`) と2つの属性を持つ**Smart Lamp**
は、次のようにプロビジョニングできます。

#### :one::zero: リクエスト:

```console
curl -iX POST \
  'http://localhost:4041/iot/devices' \
  -H 'Content-Type: application/json' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /' \
  -d '{
  "devices": [
    {
      "device_id": "lamp001",
      "entity_name": "urn:ngsi-ld:Lamp:001",
      "entity_type": "Lamp",
      "transport": "HTTP",
      "endpoint": "http://iot-sensors:3001/iot/lamp001",
      "commands": [
        {"name": "on","type": "command"},
        {"name": "off","type": "command"}
       ],
       "attributes": [
        {"object_id": "s", "name": "state", "type":"Text"},
        {"object_id": "l", "name": "luminosity", "type":"Integer"}
       ],
       "static_attributes": [
         {"name":"refStore", "type": "Relationship","value": "urn:ngsi-ld:Store:001"}
      ]
    }
  ]
}
'
```

プロビジョニングされたデバイスの完全なリストは、`/iot/devices`
エンドポイントに GET リクエストを行うことで取得できます。

#### :one::one: リクエスト:

```console
curl -X GET \
  'http://localhost:4041/iot/devices' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /'
```

<a name="enabling-context-broker-commands"></a>

## Context Broker コマンドの有効化

IoT Agent を IoT デバイスに接続すると、Orion Context Broker にコマンドが利用
可能になったことが通知されます。言い換えると、IoT Agent は、コマンド属性の
[コンテキスト・プロバイダ](https://github.com/FIWARE/tutorials.Context-Providers/)
として登録されました。

コマンドが登録されると、
[以前のチュートリアル](https://github.com/FIWARE/tutorials.IoT-Sensors)
で行ったように JSON リクエストを IoT デバイスに直接送信するのではなく、
Orion Context Broker にリクエストを送信することで、**Bell** を呼び出し、
**Smart Door** を開閉し、**Smart Lamp** のオン/オフを切り替えることができます。

IoT Agent のノース・ポートを出入りするすべての通信は、標準の NGSI 構文を使用
します。IoTデバイス と IoT Agent の間で使用されるトランスポートプロトコルは、
この通信層とは無関係です。事実上、IoT Agent は、既知のエンドポイントの単純化
された Facade パターン を提供して、あらゆるデバイスを作動させます。

したがって、コマンドの登録と呼び出しのこのセクションは、
[以前のチュートリアル](https://github.com/FIWARE/tutorials.IoT-Agent)
 にある指示を**複製**します。

<a name="ringing-the-bell"></a>

### ベルを鳴らす

`ring` コマンドを呼び出すには、コンテキストで` ring` 属性を更新する必要が
あります。

#### :one::two: リクエスト:

```console
curl -iX PATCH \
  'http://localhost:1026/v2/entities/urn:ngsi-ld:Bell:001/attrs' \
  -H 'Content-Type: application/json' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /' \
  -d '{
  "ring": {
      "type" : "command",
      "value" : ""
  }
}'
```

デバイス・モニタのページを表示している場合は、ベルの変化の状態も確認できます。

![](https://fiware.github.io/tutorials.IoT-Agent-JSON/img/bell-ring.gif)

<a name="opening-the-smart-door"></a>

### Smart Door を開く

`open` コマンドを呼び出すには、コンテキストで `open` 属性を更新する必要が
あります。

#### :one::three: リクエスト:

```console
curl -iX PATCH \
  'http://localhost:1026/v2/entities/urn:ngsi-ld:Door:001/attrs' \
  -H 'Content-Type: application/json' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /' \
  -d '{
  "open": {
      "type" : "command",
      "value" : ""
  }
}'
```
<a name="switching-on-the-smart-lamp"></a>

### Smart Lamp のスイッチをオン

**Smart Lamp** をオンにするには、コンテキストで `on` 属性を更新する必要が
あります。

#### :one::four: リクエスト:

```console
curl -iX PATCH \
  'http://localhost:1026/v2/entities/urn:ngsi-ld:Lamp:001/attrs' \
  -H 'Content-Type: application/json' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /' \
  -d '{
  "on": {
      "type" : "command",
      "value" : ""
  }
}'
```

<a name="service-group-crud-actions"></a>
<a name="creating-a-service-group"></a>
<a name="read-service-group-details"></a>
<a name="list-all-service-groups"></a>
<a name="update-a-service-group"></a>
<a name="delete-a-service-group"></a>
<a name="device-crud-actions"></a>
<a name="creating-a-provisioned-device"></a>
<a name="read-provisioned-device-details"></a>
<a name="list-all-provisioned-devices"></a>
<a name="update-a-provisioned-device"></a>
<a name="delete-a-provisioned-device"></a>


<a name="next-steps"></a>

# 次のステップ

高度な機能を追加することで、アプリケーションに複雑さを加える方法を知りたいですか
？このシリーズ
の[他のチュートリアル](https://www.letsfiware.jp/fiware-tutorials)を読むことで見
つけることができます :

---

## ライセンス

[MIT](LICENSE) © 2018-2020 FIWARE Foundation e.V.
