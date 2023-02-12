---
title: "OpenAPIのYAMLファイルをCSVに変換するCLI(Node.js)を作成した"
emoji: "🛠"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [nodejs, openapi, cli, typescript]
published: false
---

OpenAPIのYAMLファイルをCSVに変換するCLIを作成しました。
https://github.com/horitaka/openapi-yaml-to-csv-excel

CLIを開発したときの技術的な話やどういったプロセスで実施したかの記事もあわせてどうぞ。
https://zenn.dev/horitaka/articles/openapi-yaml-to-csv-excel-tech
https://zenn.dev/horitaka/articles/openapi-yaml-to-csv-excel-process



# 背景
運用チームから、APIの一覧が必要との要望を受けました。
OpenAPIの仕様に沿って記述したYAML形式のAPI仕様書はあったのですが、エクセルで管理したいとの要望がありエクセルのAPI一覧を作成することとしました。
一方で、開発は継続して続いており、API仕様書(yaml)とAPI一覧(csv)を二重でメンテナンスすることは避けたいと考えておりました。
そこで、API仕様書(yaml)からコマンドで簡単にAPI一覧(csv)を作成できるようにCLIをOSSで作りました。


# OpenAPIとは
https://spec.openapis.org/oas/latest.html

> The OpenAPI Specification (OAS) defines a standard, programming language-agnostic interface description for HTTP APIs, which allows both humans and computers to discover and understand the capabilities of a service without requiring access to source code, additional documentation, or inspection of network traffic.

# CLIの使用方法

## YAMLからCSVへの変換

Node.jsがインストールされている環境で、以下のコマンドで簡単に使用できます。
```bash
npx oapi-convert convert -i input-file.yaml -o output-file.csv
```

変換前後のファイルのサンプルは以下になります。

:::details Input file
```yaml
openapi: '3.0.0'
info:
  version: 1.0.0
  title: Swagger Petstore
  license:
    name: MIT
servers:
  - url: http://petstore.swagger.io/v1
paths:
  /pets:
    summary: Pets resources
    description: Pets resources description
    get:
      summary: List all pets
      description: List all pets description
      operationId: listPets
      tags:
        - pets
      parameters:
        - name: limit
          in: query
          description: How many items to return at one time (max 100)
          required: false
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
                $ref: '#/components/schemas/Pets'
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
    post:
      summary: Create a pet
      description: Create a pet description
      operationId: createPets
      tags:
        - pets
      responses:
        '201':
          description: Null response
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
  /pets/{petId}:
    summary: Pets resources by id
    description: Pets resources by id description
    get:
      summary: Info for a specific pet
      description: Info for a specific pet description
      operationId: showPetById
      tags:
        - pets
      parameters:
        - name: petId
          in: path
          required: true
          description: The id of the pet to retrieve
          schema:
            type: string
      responses:
        '200':
          description: Expected response to a valid request
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Pet'
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
components:
  schemas:
    Pet:
      type: object
      required:
        - id
        - name
      properties:
        id:
          type: integer
          format: int64
        name:
          type: string
        tag:
          type: string
    Pets:
      type: array
      maxItems: 100
      items:
        $ref: '#/components/schemas/Pet'
    Error:
      type: object
      required:
        - code
        - message
      properties:
        code:
          type: integer
          format: int32
        message:
          type: string
```
:::

:::details Output file
```csv
path,summary,description,method,tags,summaryMethod,descriptionMethod,operationId
/pets,Pets resources,Pets resources description,get,pets,List all pets,List all pets description,listPets
/pets,Pets resources,Pets resources description,post,pets,Create a pet,Create a pet description,createPets
/pets/{petId},Pets resources by id,Pets resources by id description,get,pets,Info for a specific pet,Info for a specific pet description,showPetById
```
:::


## 既存のCSVの更新
一度YAMLファイルからCSVファイルに変換したあとで、機能追加や改修でOpenAPIで記述された仕様書(yaml)を更新することもあると思います。
そういった場合は、差分だけを更新できたほうが便利かと思います。
このCLIではそのような機能も用意しています。

```bash
npx oapi-convert convert -i input-file.yaml -u update-file.csv -o output-file.csv
```

input-file.yamlとupdate-file.csvのoperationIdが一致するレコードを比較して、差分のみを更新するようにしています。

# まとめ
ご興味持っていただけたらぜひお使いいただけるとうれしいです。
「こんな機能もほしい！」というのがあればぜひIssueに書いてください。
気に入っていただけたらStarを押してもらえると嬉しいです
https://github.com/horitaka/openapi-yaml-to-csv-excel