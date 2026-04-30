# devpet コンポーネント依存関係

## 依存関係マトリクス

```
依存元 ＼ 依存先    | shared | cli-core | cli-ui | cli-net | backend-api | backend-svc | backend-data | backend-ext | web | cdk
--------------------|--------|----------|--------|---------|-------------|-------------|--------------|-------------|-----|----
shared              |   -    |          |        |         |             |             |              |             |     |
cli-core            |   ●    |    -     |        |         |             |             |              |             |     |
cli-ui              |   ●    |    ●     |   -    |         |             |             |              |             |     |
cli-net             |   ●    |          |        |    -    |             |             |              |             |     |
backend-api         |   ●    |          |        |         |      -      |      ●      |              |             |     |
backend-svc         |   ●    |          |        |         |             |      -      |      ●       |      ●      |     |
backend-data        |   ●    |          |        |         |             |             |      -       |             |     |
backend-ext         |        |          |        |         |             |             |              |      -      |     |
web                 |        |          |        |         |             |             |              |             |  -  |
cdk                 |        |          |        |         |             |             |              |             |     |  -

● = 依存あり
```

## データフロー図

```
+------------------------------------------------------------------+
|                        devpet システム                             |
+------------------------------------------------------------------+

[ユーザー]
    |
    | キー入力 ([t] talk, [q] quit, [h] help, [s] save)
    v
+------------------+
|  TUI Client      |
|  +-----------+   |     ローカルファイル
|  | UI Layer  |   |     (~/.devpet/)
|  +-----+-----+   |        |
|        |         |        |
|  +-----v-----+   |   +----v----+
|  | Core Layer|<--+-->| Data    |
|  | (Engine)  |   |   | Layer   |
|  +-----+-----+   |   +---------+
|        |         |
|  +-----v-----+   |
|  | Net Layer |   |
|  +-----+-----+   |
+--------+---------+
         |
         | HTTPS REST API (JSON)
         | Authorization: Bearer <JWT>
         v
+--------+---------+
|  API Gateway     |
|  (REST)          |
+--------+---------+
         |
    +----+----+----+
    |         |    |
    v         v    v
+-------+ +------+ +------+
| Talk  | | Save | | Load |
|Handler| |Handle| |Handle|
+---+---+ +--+---+ +--+---+
    |        |         |
    v        v         v
+---+--------+---------+---+
|       Service Layer       |
| +-------+ +------+ +---+ |
| |  AI   | | Pet  | |Auth| |
| |Service| |Servic| |Svc | |
| +---+---+ +--+---+ +---+ |
|     |        |            |
+-----+--------+------------+
      |        |
      v        v
+----------+ +----------+
| Bedrock  | | DynamoDB |
| (Haiku)  | | (Tables) |
+----------+ +----------+
| Prompt   |
| Mgmt     |
+----------+

+------------------+     +----------+
|  Web (Next.js)   |---->| Cognito  |
|  - Landing       |     | (Auth)   |
|  - Auth           |     +----------+
|  - Dashboard     |
|  - DeviceAuth    |
+------------------+
```

## 通信パターン

### 同期通信（リクエスト/レスポンス）

| 通信経路 | 方式 | データ形式 |
|---------|------|----------|
| TUI → API Gateway | HTTPS REST | JSON |
| API Gateway → Lambda | AWS統合 | Lambda Event |
| Lambda → DynamoDB | AWS SDK | DynamoDB Item |
| Lambda → Bedrock | AWS SDK | InvokeModel Request |
| Lambda → Bedrock Prompt Mgmt | AWS SDK | GetPrompt Request |
| Web → Cognito | Amplify SDK | OAuth2 |

### ローカル通信

| 通信経路 | 方式 | データ形式 |
|---------|------|----------|
| Core → LocalStorage | ファイルI/O | JSON |
| Core → ConfigManager | ファイルI/O | JSON |

## モジュール構成（モノレポ）

```
devpet/
├── shared/          # 共有Go module
│   ├── go.mod       # module github.com/xxx/devpet/shared
│   ├── models/
│   ├── validation/
│   └── constants/
├── cli/             # TUIクライアント Go module
│   ├── go.mod       # require github.com/xxx/devpet/shared
│   ├── cmd/
│   ├── internal/
│   │   ├── ui/
│   │   ├── core/
│   │   ├── data/
│   │   └── net/
│   └── main.go
├── backend/         # Lambda関数 Go module
│   ├── go.mod       # require github.com/xxx/devpet/shared
│   ├── cmd/
│   │   ├── talk/
│   │   ├── save/
│   │   └── load/
│   ├── internal/
│   │   ├── handler/
│   │   ├── service/
│   │   ├── repository/
│   │   └── external/
│   └── ...
├── web/             # Next.js Webサイト
│   ├── package.json
│   ├── src/
│   │   ├── app/
│   │   ├── components/
│   │   └── lib/
│   └── ...
└── cdk/             # AWS CDK (TypeScript)
    ├── package.json
    ├── lib/
    │   ├── core-stack.ts
    │   ├── service-stack.ts
    │   └── constructs/
    │       ├── database-construct.ts
    │       ├── auth-construct.ts
    │       ├── api-construct.ts
    │       ├── functions-construct.ts
    │       └── web-construct.ts
    └── bin/
        └── cdk.ts
```
