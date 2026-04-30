# devpet コンポーネント定義

## 1. TUIクライアント (devpet-cli)

### 1.1 UI Layer

| コンポーネント | 責務 |
|--------------|------|
| **MainScreen** | メイン画面。ペットAA表示、ステータスバー、吹き出し、キーバインド表示を統合 |
| **SettingsScreen** | 設定画面。認証設定（ログイン/ログアウト）、表示設定 |
| **HelpScreen** | ヘルプ画面。キーバインド一覧、コマンド説明 |
| **StatusBar** | ペットステータス表示（名前、進化段階、LV、HP、EXP） |
| **ParameterDisplay** | パラメータ表示（かしこさ/きずな/きぶん/たいりょく） |
| **PetView** | ペットAAキャラクター描画（進化段階に応じたAA切り替え） |
| **SpeechBubble** | 吹き出し表示（AI応答テキスト） |
| **DeathScreen** | 死亡演出画面（墓石表示 + 再スタート案内） |

### 1.2 Core Layer

| コンポーネント | 責務 |
|--------------|------|
| **PetEngine** | ペットの状態管理。パラメータ計算、レベルアップ判定、進化判定、死亡判定 |
| **TimeCalculator** | 経過時間計算。前回操作時刻からのパラメータ自然変動・死亡判定をローカルで計算 |
| **ExpCalculator** | EXP計算。話しかけ・AIツール連携のEXP獲得量を計算 |
| **EvolutionManager** | 進化管理。パラメータ傾向に基づく進化先の判定、進化段階の管理 |

### 1.3 Data Layer

| コンポーネント | 責務 |
|--------------|------|
| **LocalStorage** | ローカルデータ永続化。`~/.devpet/` にJSON形式で保存/読み込み |
| **ConfigManager** | 設定管理。認証トークン、表示設定等の管理 |

### 1.4 Network Layer

| コンポーネント | 責務 |
|--------------|------|
| **APIClient** | AWSバックエンドとのHTTP通信。REST APIの呼び出し、エラーハンドリング |
| **AuthClient** | Cognito認証フロー。デバイスコード認証、トークン管理、リフレッシュ |

---

## 2. AWSバックエンド (devpet-backend)

### 2.1 API Layer (Lambda Functions)

| コンポーネント | 責務 |
|--------------|------|
| **TalkHandler** | 「話しかける」API。ペット状態を受け取り、AI応答を生成して返却 |
| **SaveHandler** | セーブAPI。ローカルのペットデータをDynamoDBに保存 |
| **LoadHandler** | ロードAPI。DynamoDBからペットデータを取得して返却 |
| **AuthHandler** | 認証関連API。デバイスコード認証フローのサポート |

### 2.2 Service Layer

| コンポーネント | 責務 |
|--------------|------|
| **AIService** | AI応答生成。Bedrock Prompt Managementからプロンプト取得、パラメータ埋め込み、Claude Haiku呼び出し |
| **PetService** | ペットデータ管理。CRUD操作、データバリデーション |
| **AuthService** | 認証サービス。Cognitoとの連携、トークン検証 |

### 2.3 Data Layer

| コンポーネント | 責務 |
|--------------|------|
| **PetRepository** | DynamoDB操作。ペットデータの読み書き（生死ステータス含む） |
| **UserRepository** | DynamoDB操作。ユーザーデータの読み書き |

### 2.4 External Integration

| コンポーネント | 責務 |
|--------------|------|
| **BedrockClient** | Amazon Bedrock Runtime呼び出し。InvokeModel API |
| **PromptClient** | Bedrock Prompt Management呼び出し。GetPrompt API |
| **CognitoClient** | Amazon Cognito呼び出し。トークン検証、ユーザー管理 |

---

## 3. 紹介Webサイト (devpet-web)

### 3.1 Pages

| コンポーネント | 責務 |
|--------------|------|
| **LandingPage** | トップページ。devpet紹介、スクリーンショット、ダウンロード案内 |
| **AuthPage** | 認証ページ。サインアップ/サインイン（Cognito連携） |
| **DashboardPage** | ダッシュボード。ログイン後のアカウント管理、セーブデータ確認 |
| **DeviceAuthPage** | デバイス認証ページ。TUIからのデバイスコード認証フローを処理 |

### 3.2 Components

| コンポーネント | 責務 |
|--------------|------|
| **AuthProvider** | 認証状態管理。Cognito Amplify UIとの統合 |
| **DownloadSection** | ダウンロードセクション。OS検出、インストールコマンド表示 |

---

## 4. 共有パッケージ (devpet-shared)

| コンポーネント | 責務 |
|--------------|------|
| **Models** | 共有データモデル。Pet, TalkResponse, SaveRequest等の型定義 |
| **Validation** | 共有バリデーション。パラメータ範囲チェック、データ整合性検証 |
| **Constants** | 共有定数。進化段階名、パラメータ上限値、EXP計算係数等 |

---

## 5. インフラ (devpet-cdk)

### スタック構成

| スタック | 責務 |
|---------|------|
| **CoreStack** | データ基盤・認証基盤。デプロイ頻度が低い安定したリソース群 |
| **ServiceStack** | アプリケーションサービス。API、Lambda、Webホスティング。デプロイ頻度が高い |

### CoreStack — Constructs

| Construct | 責務 |
|-----------|------|
| **DatabaseConstruct** | DynamoDBテーブル定義（Pets, Users）、暗号化設定、GSI |
| **AuthConstruct** | Cognito User Pool、App Client、デバイスコード認証設定 |

### ServiceStack — Constructs

| Construct | 責務 |
|-----------|------|
| **ApiConstruct** | API Gateway定義、ルート設定、レート制限、CORS設定 |
| **FunctionsConstruct** | Lambda関数群（Talk, Save, Load, Auth）、Bedrock/DynamoDB/Cognito権限 |
| **WebConstruct** | Webサイトホスティング（CloudFront + S3 or Amplify Hosting） |
