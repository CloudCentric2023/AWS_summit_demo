# devpet ユニット定義

## ユニット一覧

| # | ユニット名 | 種別 | 言語 | 実装順序 |
|---|-----------|------|------|---------|
| 1 | **shared** | Go Module | Go | 1st |
| 2 | **cli** | TUIアプリケーション | Go | 2nd |
| 3 | **cdk** | IaCインフラ | TypeScript | 3rd |
| 4 | **backend** | Lambda関数群 | Go | 4th |
| 5 | **web** | Webサイト | TypeScript (Next.js) | 5th |

## 実装順序の根拠

**TUI優先アプローチ**: ローカルで動作するTUIを先に構築し、バックエンド接続は後から追加する。

```
shared (1st) → cli (2nd) → cdk (3rd) → backend (4th) → web (5th)
```

1. **shared**: 全Goユニットが依存する型定義。最初に確定する必要がある
2. **cli**: TUIクライアント。ローカルモードで単独動作可能。ペット育成のコア体験を早期に検証できる
3. **cdk**: AWSインフラ。backendのデプロイ先を先に構築
4. **backend**: Lambda関数群。cdkでデプロイされたインフラ上で動作
5. **web**: 紹介Webサイト。他の全ユニットが完成した後に構築

## コンストラクション実行方式

**段階まとめ方式**: 全ユニットの設計をまとめて行い、その後全ユニットのコード生成をまとめて行う。

```
全ユニット機能設計 → 全ユニットNFR要件 → 全ユニットNFR設計
→ 全ユニットインフラ設計 → 全ユニットコード生成 → ビルド&テスト
```

---

## ユニット詳細

### Unit 1: shared

| 項目 | 内容 |
|------|------|
| **責務** | TUIクライアントとバックエンドで共有する型定義、バリデーション、定数 |
| **成果物** | Go module (`shared/`) |
| **主要コンポーネント** | Models, Validation, Constants |
| **外部依存** | なし（標準ライブラリのみ） |
| **デプロイ** | ライブラリとして他ユニットにインポートされる（単独デプロイなし） |

### Unit 2: cli

| 項目 | 内容 |
|------|------|
| **責務** | TUIペット育成クライアント。ペット表示、パラメータ管理、ローカル保存、API通信 |
| **成果物** | Go バイナリ (`cli/`) |
| **主要コンポーネント** | UI Layer (MainScreen, SettingsScreen, HelpScreen, StatusBar, ParameterDisplay, PetView, SpeechBubble, DeathScreen), Core Layer (PetEngine, TimeCalculator, ExpCalculator, EvolutionManager), Data Layer (LocalStorage, ConfigManager), Network Layer (APIClient, AuthClient) |
| **外部依存** | shared, bubbletea, bubbles, lip gloss, aws-sdk-go-v2 (Cognito) |
| **デプロイ** | クロスプラットフォームバイナリ（Linux, macOS, Windows）。npm / GitHub Releases |

### Unit 3: cdk

| 項目 | 内容 |
|------|------|
| **責務** | AWSインフラのIaC定義。全AWSリソースのプロビジョニング |
| **成果物** | CDK TypeScript プロジェクト (`cdk/`) |
| **主要コンポーネント** | CoreStack (DatabaseConstruct, AuthConstruct), ServiceStack (ApiConstruct, FunctionsConstruct, WebConstruct) |
| **外部依存** | aws-cdk-lib, constructs |
| **デプロイ** | `cdk deploy` でAWSにデプロイ |

### Unit 4: backend

| 項目 | 内容 |
|------|------|
| **責務** | AWSバックエンドAPI。AI応答生成、データ管理、認証 |
| **成果物** | Lambda関数群 Go バイナリ (`backend/`) |
| **主要コンポーネント** | API Layer (TalkHandler, SaveHandler, LoadHandler, AuthHandler), Service Layer (AIService, PetService, AuthService), Data Layer (PetRepository, UserRepository), External Integration (BedrockClient, PromptClient, CognitoClient) |
| **外部依存** | shared, aws-sdk-go-v2 (DynamoDB, Bedrock, Cognito), aws-lambda-go |
| **デプロイ** | CDK経由でLambda関数としてデプロイ |

### Unit 5: web

| 項目 | 内容 |
|------|------|
| **責務** | devpet紹介Webサイト。ダウンロード案内、Cognito認証UI、アカウント管理 |
| **成果物** | Next.js Webアプリケーション (`web/`) |
| **主要コンポーネント** | Pages (LandingPage, AuthPage, DashboardPage, DeviceAuthPage), Components (AuthProvider, DownloadSection) |
| **外部依存** | next, react, @aws-amplify/auth |
| **デプロイ** | CDK経由でCloudFront + S3 or Amplify Hosting |

---

## コード構成（モノレポ）

```
devpet/
├── shared/                 # Unit 1: 共有Go module
│   ├── go.mod
│   ├── models/
│   │   └── pet.go
│   ├── validation/
│   │   └── validate.go
│   └── constants/
│       └── constants.go
├── cli/                    # Unit 2: TUIクライアント
│   ├── go.mod
│   ├── main.go
│   ├── cmd/
│   └── internal/
│       ├── ui/
│       ├── core/
│       ├── data/
│       └── net/
├── backend/                # Unit 4: Lambda関数群
│   ├── go.mod
│   ├── cmd/
│   │   ├── talk/
│   │   ├── save/
│   │   └── load/
│   └── internal/
│       ├── handler/
│       ├── service/
│       ├── repository/
│       └── external/
├── web/                    # Unit 5: Next.js Webサイト
│   ├── package.json
│   └── src/
│       ├── app/
│       ├── components/
│       └── lib/
├── cdk/                    # Unit 3: AWS CDK
│   ├── package.json
│   ├── bin/
│   │   └── cdk.ts
│   └── lib/
│       ├── core-stack.ts
│       ├── service-stack.ts
│       └── constructs/
│           ├── database-construct.ts
│           ├── auth-construct.ts
│           ├── api-construct.ts
│           ├── functions-construct.ts
│           └── web-construct.ts
├── go.work                 # Go workspace（マルチモジュール管理）
├── package.json            # ルートpackage.json（npm workspaces）
└── README.md
```
