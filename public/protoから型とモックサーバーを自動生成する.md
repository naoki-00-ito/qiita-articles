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

`proto` 配下に作成

```proto
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
