# devpet サービス定義

## サービス一覧

### 1. AI応答サービス (AIService)

| 項目 | 内容 |
|------|------|
| **責務** | ペットの状態に基づいてAI応答を生成する |
| **入力** | ペットデータ（パラメータ、進化段階、レベル） |
| **出力** | AI応答テキスト、EXP獲得量、パラメータ変動 |
| **依存** | Bedrock Prompt Management, Bedrock Runtime (Claude Haiku) |
| **呼び出し元** | TalkHandler (Lambda) |

**オーケストレーション:**
1. ペットの進化段階に基づいてBedrock Prompt Managementからプロンプトテンプレートを取得
2. ペットのパラメータ値をプロンプト変数に埋め込み
3. Bedrock Runtime (Claude Haiku) にInvokeModelで応答生成
4. 応答テキストとパラメータ変動を返却

---

### 2. ペットデータサービス (PetService)

| 項目 | 内容 |
|------|------|
| **責務** | ペットデータのCRUD操作とビジネスルール適用 |
| **入力** | ユーザーID、ペットデータ |
| **出力** | 保存結果、ペットデータ |
| **依存** | DynamoDB (PetRepository) |
| **呼び出し元** | SaveHandler, LoadHandler (Lambda) |

**オーケストレーション:**
- **Save**: バリデーション → DynamoDB書き込み
- **Load**: DynamoDB読み込み → データ返却
- **MarkDead**: ペットの isDead=true, diedAt を設定 → DynamoDB更新

---

### 3. 認証サービス (AuthService)

| 項目 | 内容 |
|------|------|
| **責務** | ユーザー認証とトークン管理 |
| **入力** | 認証リクエスト（デバイスコード、トークン） |
| **出力** | 認証トークン、認証状態 |
| **依存** | Amazon Cognito |
| **呼び出し元** | AuthHandler (Lambda), AuthClient (TUI), AuthProvider (Web) |

**認証フロー（デバイスコード認証）:**
1. TUIが `devpet login` を実行
2. バックエンドがCognitoにデバイスコードを発行リクエスト
3. TUIがブラウザを開き、Webサイトのデバイス認証ページに遷移
4. ユーザーがWebサイトでログイン（またはサインアップ）
5. ユーザーがデバイスコードを確認・承認
6. TUIがポーリングで認証完了を検知
7. アクセストークン + リフレッシュトークンをローカルに保存

---

## サービス間通信パターン

### TUI → バックエンド

| 通信 | プロトコル | 認証 | エラー処理 |
|------|----------|------|-----------|
| 話しかけ | HTTPS REST (POST) | Cognito JWT | エラー時ローカルモードにフォールバック |
| セーブ | HTTPS REST (PUT) | Cognito JWT | エラー表示、ローカルデータは保持 |
| ロード | HTTPS REST (GET) | Cognito JWT | エラー時ローカルデータを使用 |
| 認証 | HTTPS REST | なし（認証フロー自体） | エラー表示、未認証状態を維持 |

### バックエンド → AWSサービス

| 通信 | サービス | プロトコル |
|------|---------|----------|
| AI応答生成 | Bedrock Runtime | AWS SDK (InvokeModel) |
| プロンプト取得 | Bedrock Prompt Management | AWS SDK (GetPrompt) |
| データ読み書き | DynamoDB | AWS SDK (PutItem/GetItem/DeleteItem) |
| トークン検証 | Cognito | AWS SDK (JWT検証) |

### Web → AWSサービス

| 通信 | サービス | プロトコル |
|------|---------|----------|
| 認証 | Cognito | Amplify Auth SDK |
| デバイス認証承認 | バックエンド API | HTTPS REST |
