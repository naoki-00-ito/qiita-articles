---
title: protoから型とモックサーバーを自動生成する
tags:
  - ''
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
## Next.js プロジェクト作成

```bash
npx create-next-app@latest . --use-pnpm
```

## Protobuf 設定

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

`proto/example/example.proto` を作成

```proto
syntax = "proto3";

package example;

import "google/protobuf/timestamp.proto";
import "google/api/annotations.proto";

service ExampleService {
  rpc Example(ExampleRequest) returns (ExampleResponse) {};
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

## ts-protoc-gen をインストール

```bash
pnpm add -D ts-protoc-gen
```

## openapi 生成設定

https://buf.build/grpc-ecosystem/openapiv2?version=v2.26.3

`buf.gen.yaml` に以下を追加

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

`npx buf generate` を実行すると、`gen` ディレクトリに go と openapi のファイルが生成されます。

openapi のファイルは以下のような内容で生成されます。

```json
{
  "swagger": "2.0",
  "info": {
    "title": "example/example.proto",
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
    "exampleExample": {
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
    "exampleExampleResponse": {
      "type": "object",
      "properties": {
        "example": {
          "$ref": "#/definitions/exampleExample"
        }
      }
    },
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

package example;

import "google/protobuf/timestamp.proto";
+import "google/api/annotations.proto";

service ExampleService {
  rpc Example(ExampleRequest) returns (ExampleResponse) {
+   option (google.api.http) = {
+     get: "/example/get/{id}"
+   };
  };
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
