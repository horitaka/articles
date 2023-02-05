---
title: "OpenAPI仕様書とPrismを使ってコマンドひとつでお手軽にモックサーバーをたてる"
emoji: "🪗"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [openapi, swagger, mock, prism]
published: true
---

OpenAPI仕様書と[Prism](https://docs.stoplight.io/docs/prism/674b27b261c3c-overview)のDockerイメージを使ってコマンドひとつで簡単にモックサーバーをたてる方法です。

**動作確認をしたバージョン**
- OpenAPI: 3.0.0
- Prism: 4.5.0

**API仕様書のサンプル**
[petstore](https://github.com/OAI/OpenAPI-Specification/blob/main/examples/v3.0/petstore.yaml)のサンプルを使って確認をしました。


# docker コンテナを起動してコマンドを実行する
API仕様書が置いてあるディレクトリで以下のコマンドを実行します。
```bash
docker run --rm -it -p 4010:4010 -v $PWD:/tmp stoplight/prism:4 mock -h 0.0.0.0 /tmp/api.yml
```

これだけで簡単にモックサーバーがたてられます。
```bash
[1:33:16 AM] › [CLI] …  awaiting  Starting Prism…
[1:33:17 AM] › [CLI] ℹ  info      GET        http://0.0.0.0:4010/pets?limit=-1094358341
[1:33:17 AM] › [CLI] ℹ  info      POST       http://0.0.0.0:4010/pets
[1:33:17 AM] › [CLI] ℹ  info      GET        http://0.0.0.0:4010/pets/quos
[1:33:17 AM] › [CLI] ▶  start     Prism is listening on http://0.0.0.0:4010
```

例えば、GETを実行するとレスポンスが返ってきます。
```bash
curl localhost:4010/pets 
{"code":-2147483648,"message":"string"}
```

# 便利なところ&いろいろな活用方法

## Auto reload
Auto reloadにも対応しているため、API仕様書を修正しながらの開発も簡単です。
https://docs.stoplight.io/docs/prism/beeaad4dc0227-prism-cli#auto-reloading

> Prism watches for changes made to the document it was loaded with. When changes happen, Prism restarts its HTTP server to reflect changes to operations. There is no need to manually stop and start a Prism server after a change to a specification file.


## docker composeでFrontendとモックサーバーを同時に起動
Frontendのローカル開発をDockerコンテナで行っているのであれば、docker composeにFrontendのローカル起動とモックサーバー起動をまとめておくことで、`docker compose up`のみでFrontendとモックサーバーの両方が起動できます。

**ディレクトリ構成**
```bash
.
├── api.yaml
├── docker-compose.yaml
└── src
    ├── index.js
    └── ...
```

**docker-compose.yaml**
```yaml:docker-compose.yaml
version: '3.9'
services:
  frontend:
    ...
  prism:
    image: stoplight/prism:4
    command: 'mock -h 0.0.0.0 /tmp/api.yaml'
    volumes:
      - ./api.yml:/tmp/api.yaml:ro
    ports:
      - '4010:4010'
```

## レスポンスを動的に変更する
`-d`オプションをつけることでレスポンスを動的に切り替えることができます。
```bash
docker run --rm -it -p 4010:4010 -v $PWD:/tmp stoplight/prism:4 mock -h 0.0.0.0 /tmp/api.yaml -d
```

https://docs.stoplight.io/docs/prism/9528b5a8272c0-dynamic-response-generation-with-faker


## パラメーターのバリデーション
リクエストパラメーターのバリデーションも行ってくれます。
API仕様書で400を定義しておけば、400 Bad requestとしてレスポンスを返してくれます。

例えば、PetstoreのAPIを以下のように変更して、
```diff
paths:
  /pets:
    get:
      summary: List all pets
      operationId: listPets
      tags:
        - pets
      parameters:
        - name: limit
          in: query
          description: How many items to return at one time (max 100)
-         required: false
+         required: true
          schema:
            type: integer
            maximum: 100
            format: int32
      responses:
        '200':
          description: A paged array of pets
          headers:
            x-next:
              description: A link to the next page of responses
              schema:
                type: string
          content:
            application/json:    
              schema:
                $ref: "#/components/schemas/Pets"
+       400:
+         description: unexpected error
+         content:
+           application/json:
+             schema:
+               $ref: "#/components/schemas/Error"
```

limitパラメーターなしでGETを実行すると400 Bad Requestでエラーが返ってきていることがわかります。
```bash
curl -D - localhost:4010/pets                                                               
HTTP/1.1 400 Bad Request
```

尚、API仕様書で400が定義されていない場合は、422でレスポンスが返ってくるようになっています。
> In case there's neither a 422 nor a 400 defined, Prism will create a 422 response code for you indicating the validation errors it found along the way. Such response will have a payload conforming to the application/problem+json specification.

https://docs.stoplight.io/docs/prism/21adcc8dc5fe5-request-validation


## CORSの設定
デフォルトではCORSの設定は有効化されています。
CORSを無効化したい場合は`--cors=false`オプションをつけることで可能です。

https://docs.stoplight.io/docs/prism/5ee9bb8a1cf2b-cors

## Postman Collectionを使用する
Postman Collectionを使用してモックサーバーをたてることもできます。
```bash
docker run --rm -it -p 4010:4010 -v $PWD:/tmp stoplight/prism:4 mock -h 0.0.0.0 /tmp/postman-collection.json
```

https://docs.stoplight.io/docs/prism/bffdc5c112ca9-postman-collections-support

