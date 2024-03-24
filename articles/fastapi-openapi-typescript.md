---
title: "FastAPIで定義したschemaからOpenAPI仕様書を出力し、TypeScriptの型情報まで自動生成する"
emoji: "🛣️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [fastapi,typescript, openapi, chatgpt, python]
published: true
---

# 概要
Frontend開発でAPIクライアントのリクエストやレスポンス情報の型定義をすることがあると思います。
Backend(Python, FastAPI)で定義したAPI仕様を元に自動で生成されるようにし、BackendとFrontendで型の不一致が起きないようにしました。
TypeScriptの型情報を自動で生成するライブラリはいくつかありますが、今回はリクエストとレスポンスのモデル情報さえあれば良かったため、ChatGPTの力も借りつつ、自前でスクリプトを作成しました。

**環境情報**
- Python: 3.11
- FastAPI: 0.110.0

**サンプルコード**
この記事で紹介しているコードは下記にあります。
https://github.com/horitaka/fastapi-openapi-typescript

# 構成
![FastAPIのスキーマ定義からTypeScriptの型情報を自動生成するシステム](https://storage.googleapis.com/zenn-user-upload/fc3824101dc6-20240324.jpg)

# コード

## 1. FastAPIのAPI定義からのOpenAPI仕様書の生成

### API仕様の定義

FastAPIではAPIのエンドポイントの関数で定義したschemaを元にして、OpenAPIに準拠した仕様書を生成することができます。
はじめに、schemaを定義します。

```py
from typing import List
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class User(BaseModel):
    id: int
    name: str
    email: str
    address: str | None = None


class Users(BaseModel):
    users: List[User] = []


class GetUsersResponse(BaseModel):
    users: List[User] = []


@app.get("/users")
async def get_users(limit: int = 20, name: str | None = None) -> GetUsersResponse:
    return GetUsersResponse(users=[])

```

### OpenAPI仕様書の出力

FastAPIクラスのインスタンスから`openapi()`を実行するのみで、OpenAPI仕様に準拠したテキストの生成ができます。
これをそのままファイルに保存すれば、OpenAPIの仕様に準拠したAPI仕様書が生成されます。

```py
from main import app
import json

with open("openapi.json", "w") as f:
    api_spec = app.openapi()
    f.write(json.dumps(api_spec))
```

**参考資料**
[FastAPI - Extending OpenAPI](https://fastapi.tiangolo.com/how-to/extending-openapi/)

## 2. OpenAPI仕様書からTypeScriptの型定義を出力

OpenAPI仕様書からTypeScriptの型定義を出力するプログラムの紹介です。
同じようなことが実現できるライブラリは多数ありますので、要件を満たすものであればそれを使用しても良いと思います。
これらにはTypeScript以外の言語にも対応していたり、APIクライアントの生成まで行えるものがあります。
今回は、TypeScriptの型定義を生成するのみという単純なものであったため、自前でスクリプトを作成しました。

### 初期バージョン
ChatGPTを使用して作成します。
参考に使用したプロンプトを以下に掲載します。

:::details プロンプト
OpenAPIの仕様書から、components部分のschemaのtypescriptの型定義ファイルを生成するpythonスクリプトを書いてください

## 入力例
```json
{
  "openapi": "3.1.0",
  "info": { "title": "FastAPI", "version": "0.1.0" },
  "paths": {
    "/users": {
      "get": {
        "summary": "Get Users",
        "operationId": "get_users_users_get",
        "parameters": [
          {
            "name": "limit",
            "in": "query",
            "required": false,
            "schema": { "type": "integer", "default": 20, "title": "Limit" }
          },
          {
            "name": "name",
            "in": "query",
            "required": false,
            "schema": {
              "anyOf": [{ "type": "string" }, { "type": "null" }],
              "title": "Name"
            }
          }
        ],
・・・ (実際のプロンプトでは省略せずに記載しています)
```

## 出力例
```ts
type User = {
  id: number;
  name: string;
  email: string;
};

type GetUsersResponse = {
  users: User[];
};
(省略)
```
:::


以下のようなPythonコードが生成されました。

:::details 生成されたコード
```py
import json

def generate_ts_types(openapi_json):
    components = openapi_json['components']['schemas']
    ts_types = []

    def json_type_to_ts(json_type, prop_details=None):
        mapping = {
            'string': 'string',
            'integer': 'number',
            'object': 'any',  # You may want to handle nested objects more precisely.
            'array': 'Array<any>',  # You may want to handle arrays of specific types.
            'boolean': 'boolean',
        }

        if json_type == 'array' and prop_details and 'items' in prop_details:
            item_type = prop_details['items'].get('type', 'any')
            if '$ref' in prop_details['items']:
                ref = prop_details['items']['$ref']
                item_type = ref.split('/')[-1]  # Assuming ref format is #/components/schemas/TypeName
            return f'Array<{json_type_to_ts(item_type)}>'

        if json_type in mapping:
            return mapping[json_type]
        return 'any'

    for name, schema in components.items():
        if schema.get('type') == 'object':
            ts_type = f"type {name} = {{\n"
            for prop_name, prop_details in schema.get('properties', {}).items():
                prop_type = prop_details.get('type', 'any')
                if '$ref' in prop_details:
                    ref = prop_details['$ref']
                    prop_type = ref.split('/')[-1]  # Assuming ref format is #/components/schemas/TypeName
                elif 'anyOf' in prop_details:
                    prop_type = ' | '.join([json_type_to_ts(p['type']) if 'type' in p else p['$ref'].split('/')[-1] for p in prop_details['anyOf']])
                else:
                    prop_type = json_type_to_ts(prop_type, prop_details)
                ts_type += f"  {prop_name}: {prop_type};\n"
            ts_type += "};\n"
            ts_types.append(ts_type)
        elif schema.get('type') == 'string' and 'enum' in schema:
            enum_values = ' | '.join([f"'{value}'" for value in schema['enum']])
            ts_type = f"type {name} = {enum_values};\n"
            ts_types.append(ts_type)

    return '\n'.join(ts_types)

# Assume you load the JSON from a file or API response
# with open('path_to_your_openapi_json_file.json', 'r') as file:
#     openapi_json = json.load(file)

openapi_json = {
    # Paste the JSON example here
}

print(generate_ts_types(openapi_json))
```
:::


### 改善
生成されたコードには何点か改善が必要な点があったため、順番に解決しました。
改善した内容は具体的なプロンプトを見ていただく方が早いと思いますので、そのままご覧ください。


:::details 改善1
以下の点を修正してほしいです。

1. `type`ではなく、`export type`で出力してください
2. `Array<any>`ではなく、`any[]`の形式で出力してください
3. anyでは出力しないでください。代わりに具体的な型を生成してください
:::

:::details 改善2
Userの内容に誤りがあります。スクリプトの修正をお願いします

##誤った出力
```ts
export type User = {
  id: number;
  name: string;
  email: string;
  address?: string | any;
};
```

##正しい出力
```ts
export type User = {
  id: number;
  name: string;
  email: string;
  address?: string | null;
};
```
:::


### 最終版
TypeScriptの型情報を出力するコードの最終盤は下記です。

:::details 最終版
```ts:openapi_to_ts.py
from main import app


def generate_ts_types(openapi_json):
    components = openapi_json["components"]["schemas"]
    ts_types = []
    mapping = {
        "string": "string",
        "integer": "number",
        "boolean": "boolean",
    }

    def resolve_ref(ref):
        return ref.split("/")[-1]

    def json_type_to_ts(json_type, prop_details=None):

        if json_type == "array" and prop_details:
            if "items" in prop_details:
                item_type = "any"
                if "type" in prop_details["items"]:
                    item_type = prop_details["items"]["type"]
                    item_type = mapping.get(
                        item_type, item_type
                    )  # Use direct type or map
                if "$ref" in prop_details["items"]:
                    item_type = resolve_ref(prop_details["items"]["$ref"])
                if "anyOf" in prop_details["items"]:
                    item_types = [p.get("type") for p in prop_details["items"]["anyOf"]]
                    item_types = [mapping.get(t, t) for t in item_types]  # Map types
                    item_type = " | ".join(item_types)
                return f"({item_type})[]"
            else:
                return "any[]"

        return mapping.get(json_type, "any")

    for name, schema in components.items():
        if schema.get("type") == "object":
            ts_type = f"export type {name} = {{\n"
            for prop_name, prop_details in schema.get("properties", {}).items():
                is_required = prop_name in schema.get("required", [])
                prop_type = "any"
                if "$ref" in prop_details:
                    prop_type = resolve_ref(prop_details["$ref"])
                elif "anyOf" in prop_details:
                    types = [p.get("type") for p in prop_details["anyOf"]]
                    types = [
                        mapping.get(t, t) for t in types if t is not None
                    ]  # Map and filter None
                    prop_type = " | ".join(types)
                else:
                    prop_type = json_type_to_ts(prop_details.get("type"), prop_details)
                ts_type += f"  {prop_name}{'' if is_required else '?'}: {prop_type};\n"
            ts_type += "};\n"
            ts_types.append(ts_type)
        elif schema.get("type") == "string" and "enum" in schema:
            enum_values = " | ".join([f"'{value}'" for value in schema["enum"]])
            ts_type = f"export type {name} = {enum_values};\n"
            ts_types.append(ts_type)

    return "\n".join(ts_types)


if __name__ == "__main__":
    openapi_json = app.openapi()
    ts_types = generate_ts_types(openapi_json)
    with open("api.d.ts", "w") as f:
        f.write(ts_types)
```
:::

# TypeScriptの型情報の生成結果
型情報の生成結果は以下のようになりました。

```ts
export type GetUsersResponse = {
  users?: (User)[];
};

export type HTTPValidationError = {
  detail?: (ValidationError)[];
};

export type PostUsersRequest = {
  name: string;
  email: string;
};

export type User = {
  id: number;
  name: string;
  email: string;
  address?: string | null;
};
```
