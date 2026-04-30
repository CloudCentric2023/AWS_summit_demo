# cdk — ビジネスロジックモデル

## 概要
CDKユニットはインフラ定義のみで、アプリケーションビジネスロジックは持たない。
ここでは「インフラ構成のビジネスルール」を定義する。

## 1. スタック構成

### CoreStack
デプロイ頻度が低い安定したリソース群。

| Construct | リソース | 備考 |
|-----------|---------|------|
| DatabaseConstruct | DynamoDB Pets テーブル | PK: userId, SK: petId |
| DatabaseConstruct | DynamoDB Users テーブル | PK: userId |
| AuthConstruct | Cognito User Pool | メール/パスワード認証 |
| AuthConstruct | Cognito App Client | TUI用、Web用 |

### ServiceStack
デプロイ頻度が高いアプリケーションサービス。

| Construct | リソース | 備考 |
|-----------|---------|------|
| ApiConstruct | API Gateway REST API | リソースベースURL |
| FunctionsConstruct | Lambda: Talk | POST /pets/{petId}/talk |
| FunctionsConstruct | Lambda: Save | PUT /pets/{petId}/save |
| FunctionsConstruct | Lambda: Load | GET /pets/{petId} |
| FunctionsConstruct | Lambda: Auth | POST /auth/device, POST /auth/device/poll |
| WebConstruct | CloudFront + S3 | Next.js静的ホスティング |

## 2. DynamoDB テーブル設計

### Pets テーブル
```
テーブル名: devpet-pets
PK: userId (String)
SK: petId (String)

属性:
- name (String)
- stage (String)
- evolutionType (String)
- level (Number)
- exp (Number)
- hp (Number)
- kashikosa (Number)
- kizuna (Number)
- kibun (Number)
- lastInteractionAt (String, ISO 8601)
- isDead (Boolean)
- diedAt (String, ISO 8601, optional)
- causeOfDeath (String, optional)
- createdAt (String, ISO 8601)
- updatedAt (String, ISO 8601)

GSI:
- GSI1: userId-isDead-index
  - PK: userId
  - SK: isDead
  - 用途: アクティブなペット（isDead=false）の高速検索
```

### Users テーブル
```
テーブル名: devpet-users
PK: userId (String)

属性:
- email (String)
- createdAt (String, ISO 8601)
- lastLoginAt (String, ISO 8601)
```

## 3. リソース命名規則

```
{プロジェクト名}-{環境}-{リソース種別}
例: devpet-prod-pets-table
    devpet-prod-api
    devpet-prod-talk-function
```

## 4. タグ付けルール

全リソースに以下のタグを付与:

| タグキー | 値 |
|---------|-----|
| Project | devpet |
| Environment | dev / staging / prod |
| ManagedBy | cdk |

---

## テスト可能プロパティ（PBT-01）

| コンポーネント | プロパティカテゴリ | プロパティ説明 |
|--------------|-----------------|-------------|
| CDK Stacks | Invariant | スナップショットテスト: CloudFormationテンプレートが意図しない変更を含まない |
| CDK Stacks | Invariant | 全リソースに必須タグ（Project, Environment, ManagedBy）が付与されている |
| DatabaseConstruct | Invariant | DynamoDBテーブルに暗号化設定が有効 |
| AuthConstruct | Invariant | Cognito User Poolにパスワードポリシーが設定されている |
| FunctionsConstruct | Invariant | Lambda関数に最小権限のIAMロールが付与されている |

> **Note**: CDKのPBTは主にスナップショットテストとして実装。プロパティベースの入力生成は限定的だが、構成の不変条件（invariant）を検証する。
