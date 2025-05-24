---
title: protoから生成したAPIドキュメントをもとにモックサーバーを自動生成する
tags:
  - protobuf
  - モック
  - 自動生成
  - OpenAPI
  - msw
private: false
updated_at: '2025-05-24T15:42:08+09:00'
id: 1db7e5ad34f42e3000eb
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

みなさんはAPIの仕様書をどのように作成していますか？
私は最近だと、ProtobufからOpenAPIの仕様書を生成するケースが多いです。
今回は、ProtobufからOpenAPIの仕様書を生成し、その仕様書をもとにモックサーバーを自動生成するまでの流れを紹介します。

## 前提

Node.jsを使用した、フロント用のリポジトリ内にprotoファイルを配置し、そこからOpenAPIの仕様書を生成することを想定しています。
また、モックサーバーには、[MSW](https://mswjs.io/)を使用します。

## Protobuf 設定

まず、Protobufを使うための設定を行います。
公式ドキュメントの手順に従って設定を行います。

https://buf.build/docs/cli/installation/#npm

https://buf.build/docs/cli/quickstart/

### Buf CLI インストール

```bash
pnpm add -D @bufbuild/buf
```

### Buf CLI 初期化

```bash
npx buf config init
```

### protoファイルのディレクトリ指定を追加

```diff
# For details on buf.yaml configuration, visit https://buf.build/docs/configuration/v2/buf-yaml
version: v2
+modules:
+ - path: proto
lint:
  use:
    - STANDARD
breaking:
  use:
    - FILE

```

### ビルド設定ファイル作成

```bash
touch buf.gen.yaml
```

```yaml
version: v2
managed:
  enabled: true
  override:
    - file_option: go_package_prefix
      value: github.com/bufbuild/buf-tour/gen
plugins:
  - remote: buf.build/protocolbuffers/go
    out: gen
    opt: paths=source_relative
  - remote: buf.build/connectrpc/go
    out: gen
    opt: paths=source_relative
inputs:
  - directory: proto

```

### protoファイル作成

`/proto/example/v1/example.proto` を作成します。

```proto
syntax = "proto3";

package example.v1;

import "google/protobuf/timestamp.proto";

service ExampleService {
  rpc GetExample(ExampleRequest) returns (ExampleResponse) {}
}

message Example {
  string id = 1;
  string name = 2;
  string description = 3;
  google.protobuf.Timestamp created_at = 4;
}

message ExampleRequest {
  string id = 1;
}

message ExampleResponse {
  Example example = 1;
}

```

## OpenAPIの仕様書 生成設定

https://buf.build/grpc-ecosystem/openapiv2?version=v2.26.3

`buf.gen.yaml` に以下の設定を追加します。

```diff
version: v2
managed:
  enabled: true
  override:
    - file_option: go_package_prefix
      value: github.com/bufbuild/buf-tour/gen
plugins:
  - remote: buf.build/protocolbuffers/go
    out: gen
    opt: paths=source_relative
  - remote: buf.build/connectrpc/go
    out: gen
    opt: paths=source_relative
+ - remote: buf.build/grpc-ecosystem/openapiv2:v2.26.3
+   out: gen
inputs:
  - directory: proto

```

`npx buf generate` を実行すると、`gen` ディレクトリに go と OpenAPIの仕様書が生成されます。
以下のような内容で生成されます。

```json
{
  "swagger": "2.0",
  "info": {
    "title": "example/v1/example.proto",
    "version": "version not set"
  },
  "tags": [
    {
      "name": "ExampleService"
    }
  ],
  "consumes": [
    "application/json"
  ],
  "produces": [
    "application/json"
  ],
  "paths": {},
  "definitions": {
    "protobufAny": {
      "type": "object",
      "properties": {
        "@type": {
          "type": "string"
        }
      },
      "additionalProperties": {}
    },
    "rpcStatus": {
      "type": "object",
      "properties": {
        "code": {
          "type": "integer",
          "format": "int32"
        },
        "message": {
          "type": "string"
        },
        "details": {
          "type": "array",
          "items": {
            "type": "object",
            "$ref": "#/definitions/protobufAny"
          }
        }
      }
    },
    "v1Example": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string"
        },
        "name": {
          "type": "string"
        },
        "description": {
          "type": "string"
        },
        "createdAt": {
          "type": "string",
          "format": "date-time"
        }
      }
    },
    "v1ExampleResponse": {
      "type": "object",
      "properties": {
        "example": {
          "$ref": "#/definitions/v1Example"
        }
      }
    }
  }
}

```

## gRPC Gateway 設定

### googleapis を追加

`buf.yaml` に以下を追加

```diff
# For details on buf.yaml configuration, visit https://buf.build/docs/configuration/v2/buf-yaml
version: v2
modules:
  - path: proto
lint:
  use:
    - STANDARD
breaking:
  use:
    - FILE
+deps:
+ - buf.build/googleapis/googleapis

```

以下のコマンドを実行して、googleapis を追加します。

```bash
npx buf mod update
````

### proto に REST API の設定を追加

```diff
syntax = "proto3";

package example.v1;

import "google/protobuf/timestamp.proto";
+import "google/api/annotations.proto";

service ExampleService {
  rpc GetExample(ExampleRequest) returns (ExampleResponse) {
+   option (google.api.http) = {
+     get: "/v1/examples/{id}"
+   };
  }
}

message Example {
  string id = 1;
  string name = 2;
  string description = 3;
  google.protobuf.Timestamp created_at = 4;
}

message ExampleRequest {
  string id = 1;
}

message ExampleResponse {
  Example example = 1;
}

```

`npx buf generate` で生成される OpenAPIの仕様書は以下のようになります。

```json
{
  "swagger": "2.0",
  "info": {
    "title": "example/v1/example.proto",
    "version": "version not set"
  },
  "tags": [
    {
      "name": "ExampleService"
    }
  ],
  "consumes": [
    "application/json"
  ],
  "produces": [
    "application/json"
  ],
  "paths": {
    "/v1/examples/{id}": {
      "get": {
        "operationId": "ExampleService_GetExample",
        "responses": {
          "200": {
            "description": "A successful response.",
            "schema": {
              "$ref": "#/definitions/v1ExampleResponse"
            }
          },
          "default": {
            "description": "An unexpected error response.",
            "schema": {
              "$ref": "#/definitions/rpcStatus"
            }
          }
        },
        "parameters": [
          {
            "name": "id",
            "in": "path",
            "required": true,
            "type": "string"
          }
        ],
        "tags": [
          "ExampleService"
        ]
      }
    }
  },
  "definitions": {
    "protobufAny": {
      "type": "object",
      "properties": {
        "@type": {
          "type": "string"
        }
      },
      "additionalProperties": {}
    },
    "rpcStatus": {
      "type": "object",
      "properties": {
        "code": {
          "type": "integer",
          "format": "int32"
        },
        "message": {
          "type": "string"
        },
        "details": {
          "type": "array",
          "items": {
            "type": "object",
            "$ref": "#/definitions/protobufAny"
          }
        }
      }
    },
    "v1Example": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string"
        },
        "name": {
          "type": "string"
        },
        "description": {
          "type": "string"
        },
        "createdAt": {
          "type": "string",
          "format": "date-time"
        }
      }
    },
    "v1ExampleResponse": {
      "type": "object",
      "properties": {
        "example": {
          "$ref": "#/definitions/v1Example"
        }
      }
    }
  }
}

```

## MSW をインストール

```bash
pnpm add -D msw
```

```bash
npx msw init ./public --save
```

## モック生成に使うライブラリをインストール

[公式ドキュメントでも紹介されている](https://mswjs.io/docs/recipes/keeping-mocks-in-sync/#openapi-swagger)、msw-auto-mock を使います。

https://github.com/zoubingwu/msw-auto-mock

```bash
pnpm add -D msw-auto-mock faker-js
```

## モック生成用 npm script を追加

```diff
  "scripts": {
+   "generate-mock": "npx msw-auto-mock gen/example/v1/example.swagger.json -o ./src/mocks --base-url=http://localhost:3001 --typescript"
  },
```

`pnpm generate-mock` を実行すると、`src/mocks` 配下に以下のようなファイルが生成されます。

```bash
.
└── src
    └── mocks
        ├── browser.ts
        ├── handlers.ts
        ├── native.ts
        └── node.ts
```

`src/mocks/handlers.ts` の中身をみてみると、以下のように、OpenAPIの仕様書をもとにしてモックを生成するためのコードが記載されています。

```ts
/**
 * This file is AUTO GENERATED by [msw-auto-mock](https://github.com/zoubingwu/msw-auto-mock)
 * Feel free to commit/edit it as you need.
 */
/* eslint-disable */
/* tslint:disable */
// @ts-nocheck
import { HttpResponse, http } from "msw";
import { faker } from "@faker-js/faker";

faker.seed(1);

const baseURL = "http://localhost:3001";
const MAX_ARRAY_LENGTH = 20;

// Map to store counters for each API endpoint
const apiCounters = new Map<string, number>();

const next = (apiKey: string) => {
  let currentCount = apiCounters.get(apiKey) ?? 0;
  if (currentCount === Number.MAX_SAFE_INTEGER - 1) {
    currentCount = 0;
  }
  apiCounters.set(apiKey, currentCount + 1);
  return currentCount;
};

export const handlers = [
  http.get(`${baseURL}/v1/examples/:id`, async () => {
    const resultArray = [
      [getExampleServiceGetExample200Response(), { status: 200 }],
      [getExampleServiceGetExampledefaultResponse(), { status: NaN }],
    ] as [any, { status: number }][];

    return HttpResponse.json(
      ...resultArray[next(`get /v1/examples/:id`) % resultArray.length],
    );
  }),
];

export function getExampleServiceGetExample200Response() {
  return {
    example: {
      id: faker.string.uuid(),
      name: faker.person.fullName(),
      description: faker.lorem.words(),
      createdAt: faker.date.past(),
    },
  };
}

export function getExampleServiceGetExampledefaultResponse() {
  return {
    code: faker.number.int(),
    message: faker.lorem.words(),
    details: [
      ...new Array(faker.number.int({ min: 1, max: MAX_ARRAY_LENGTH })).keys(),
    ].map((_) => ({
      "@type": faker.lorem.words(),
    })),
  };
}
```

アプリケーションから `http://localhost:3001/v1/examples/{id}` にリクエストを送ると、以下のようなレスポンスが返ってきます。

```json
{
    "id": "2040a85c-2343-41d1-aa10-8ed50ccdace7",
    "name": "Mr. Milton Lebsack",
    "description": "tremo cilicium atrox",
    "createdAt": "2025-05-06T15:31:02.034Z"
}
```

以下リポジトリに Next.js でリクエストする場合のサンプルコードも作成しています。

https://github.com/naoki-00-ito/generate-types-mock

:::note warn
公式のDraft状態のPRを参考にして作成しています。
https://github.com/mswjs/examples/pull/101

初回ロード時にモック生成に遅延が発生するなどの問題があるため、モックサーバーの自動生成ができているかの確認用途のみでの使用をお勧めします。
:::

## 参考にさせていただいた記事

https://tech.yappli.io/entry/protobuf-document
