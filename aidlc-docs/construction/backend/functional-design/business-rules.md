# backend — ビジネスルール定義

## 1. API処理ルール

### BR-BE-01: 認証必須エンドポイント
以下のエンドポイントはCognito JWT認証が必須:
- POST `/pets/{petId}/talk`
- PUT `/pets/{petId}/save`
- GET `/pets/{petId}`

### BR-BE-02: 認証不要エンドポイント
以下のエンドポイントは認証不要:
- POST `/auth/device`
- POST `/auth/device/poll`

### BR-BE-03: JWT検証ルール（SECURITY-08準拠）
1. JWTの署名を検証（Cognito公開鍵）
2. JWTの有効期限を検証
3. JWTのaudience（aud）がApp Client IDと一致することを検証
4. JWTのissuer（iss）がCognito User Pool URLと一致することを検証
5. JWTのsub claimからuserIDを抽出

### BR-BE-04: 所有権検証（SECURITY-08: IDOR防止）
- リクエストのpetIdに対応するペットのuserIdが、JWTのsubと一致することを検証
- 不一致の場合は403 Forbiddenを返却

---

## 2. AI応答ルール

### BR-BE-05: 応答長制限
- AI応答は30文字程度に制限
- プロンプトで「30文字以内の日本語で1文だけ」と指示
- 応答が長すぎる場合はトリミング（最初の50文字まで）

### BR-BE-06: プロンプトテンプレート管理
- Bedrock Prompt Managementで進化段階・進化タイプごとにテンプレートを管理
- テンプレートID命名: `devpet-{stage}-{evolutionType}-prompt`
  - 例: `devpet-egg-prompt`（たまごは進化タイプなし）
  - 例: `devpet-child-brain-prompt`, `devpet-child-heart-prompt`, `devpet-child-mood-prompt`
  - 例: `devpet-adult-sage-prompt`, `devpet-adult-trickster-prompt`, `devpet-adult-stray-prompt` 等
- テンプレート取得失敗時はデフォルトプロンプトにフォールバック
- 合計テンプレート数: 1（egg）+ 3（child）+ 7（adult）= 11

### BR-BE-07: Bedrock呼び出し設定
- モデル: Claude Haiku（anthropic.claude-3-haiku-20240307-v1:0）
- max_tokens: 100（30文字程度の日本語に十分）
- temperature: 0.8（多様性のある応答を生成）
- top_p: 0.9

---

## 3. データ管理ルール

### BR-BE-08: データ保存
- ペットデータはDynamoDBに保存
- 死亡したペットのデータは永久保持（TTLなし）
- UpdatedAtは保存時に自動更新

### BR-BE-09: データ読み込み
- GetActive: isDead=false のペットを取得（GSI使用）
- 複数のアクティブペットが存在する場合はcreatedAtが最新のものを返却
- アクティブペットが存在しない場合は404

### BR-BE-10: ユーザーデータ
- ユーザーデータはCognito認証時に自動作成
- lastLoginAtは認証成功時に更新

---

## 4. エラーハンドリングルール（SECURITY-15準拠）

### BR-BE-11: エラーレスポンス形式
```json
{
  "code": "ERROR_CODE",
  "message": "ユーザー向けの一般的なメッセージ"
}
```

### BR-BE-12: エラーコード一覧

| HTTPステータス | エラーコード | ユーザー向けメッセージ |
|--------------|------------|-------------------|
| 400 | INVALID_REQUEST | リクエストが不正です |
| 401 | UNAUTHORIZED | 認証が必要です |
| 403 | FORBIDDEN | アクセスが拒否されました |
| 404 | NOT_FOUND | リソースが見つかりません |
| 429 | RATE_LIMITED | リクエストが多すぎます。しばらくお待ちください |
| 500 | INTERNAL_ERROR | サーバーエラーが発生しました |

### BR-BE-13: エラーログ（SECURITY-03準拠）
- 全エラーを構造化ログで記録
- ログ形式: JSON
- 必須フィールド: timestamp, requestId, level, message, errorCode
- 機密情報（トークン、パスワード）はログに含めない

---

## 5. レート制限ルール（SECURITY-11準拠）

### BR-BE-14: API Gatewayレベル
- グローバル: 100リクエスト/秒
- バースト: 200リクエスト
- ユーザーあたり: 10リクエスト/秒

### BR-BE-15: Talk APIの追加制限
- Talk APIはBedrock呼び出しを伴うため、コスト管理の観点から追加制限を検討
- ユーザーあたり: 1リクエスト/秒（Talk APIのみ）
