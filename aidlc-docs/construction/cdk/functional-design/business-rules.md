# cdk — ビジネスルール定義

## 1. インフラ構成ルール

### BR-CDK-01: スタック分離
- CoreStack: データ基盤・認証基盤（デプロイ頻度低）
- ServiceStack: API・Lambda・Webホスティング（デプロイ頻度高）
- ServiceStack は CoreStack に依存（クロススタック参照）

### BR-CDK-02: DynamoDB設定
- 暗号化: AWS管理キー（AWS_OWNED）で暗号化at rest
- 課金モード: PAY_PER_REQUEST（オンデマンド）
- ポイントインタイムリカバリ: 有効
- 削除保護: 本番環境では有効

### BR-CDK-03: Lambda設定
- ランタイム: provided.al2023（Go カスタムランタイム）
- メモリ: 256MB（デフォルト）
- タイムアウト: 30秒（Talk関数はBedrock呼び出しがあるため余裕を持たせる）
- 環境変数で設定値を注入（テーブル名、Cognito設定等）

### BR-CDK-04: API Gateway設定
- REST API タイプ
- ステージ: dev / prod
- レート制限: 100リクエスト/秒（バースト: 200）
- スロットリング: ユーザーあたり10リクエスト/秒
- CORS: Webサイトのドメインのみ許可
- アクセスログ: CloudWatch Logsに出力

### BR-CDK-05: Cognito設定
- パスワードポリシー: 最小8文字、大文字・小文字・数字・記号を含む
- MFA: オプション（TOTP）
- メール検証: 必須
- セルフサインアップ: 有効

### BR-CDK-06: Webホスティング設定
- CloudFront + S3 構成
- HTTPS強制
- カスタムドメイン対応（将来）
- キャッシュ: 静的アセットは長期キャッシュ、HTMLは短期キャッシュ

---

## 2. セキュリティルール

### BR-CDK-07: 最小権限（SECURITY-06準拠）
各Lambda関数に必要最小限の権限のみ付与:

| Lambda | 必要権限 |
|--------|---------|
| Talk | DynamoDB:GetItem (Pets), Bedrock:InvokeModel, Bedrock:GetPrompt |
| Save | DynamoDB:PutItem (Pets) |
| Load | DynamoDB:GetItem, DynamoDB:Query (Pets) |
| Auth | Cognito:AdminInitiateAuth, Cognito:AdminRespondToAuthChallenge |

### BR-CDK-08: ネットワーク（SECURITY-07準拠）
- API GatewayはHTTPSのみ
- S3バケットはパブリックアクセスブロック有効
- CloudFrontのみS3にアクセス可能（OAC）

### BR-CDK-09: ログ（SECURITY-02, SECURITY-03準拠）
- API Gatewayアクセスログ: CloudWatch Logs
- Lambda実行ログ: CloudWatch Logs（自動）
- ログ保持期間: 90日
