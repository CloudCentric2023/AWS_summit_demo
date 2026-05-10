# web — ビジネスルール定義

## 1. 認証ルール

### BR-WEB-01: サインアップ
- メールアドレス + パスワードで登録
- パスワードポリシー: Cognito設定に準拠（最小8文字、大文字・小文字・数字・記号）
- メール確認コードによる検証が必須

### BR-WEB-02: サインイン
- メールアドレス + パスワードでログイン
- ログイン成功時にアクセストークン + リフレッシュトークンを取得
- トークンはAmplify SDKが自動管理（ローカルストレージ）

### BR-WEB-03: セッション管理
- アクセストークンの有効期限: Cognito設定に従う（デフォルト1時間）
- リフレッシュトークンで自動更新
- ログアウト時にトークンをクリア

---

## 2. デバイス認証ルール

### BR-WEB-04: デバイスコード承認フロー
1. URLパラメータからuser_codeを取得
2. ログイン済みでない場合 → ログインページにリダイレクト（リダイレクト先を保持）
3. ログイン済みの場合 → user_codeを表示し、承認確認
4. ユーザーが「承認」を押す → バックエンドAPIに承認リクエスト
5. 承認成功 → 「TUIでの認証が完了しました。このページを閉じてください」を表示
6. 承認失敗 → エラーメッセージを表示

### BR-WEB-05: デバイスコード有効期限
- デバイスコードの有効期限はバックエンド側で管理
- 期限切れの場合はエラーメッセージを表示し、TUIで再度ログインを促す

---

## 3. ダッシュボードルール

### BR-WEB-06: データ表示
- ログイン後、バックエンドAPIからペットデータを取得
- 表示項目（最小限）:
  - ペット名
  - レベル
  - 進化段階
- データ取得失敗時は「データを取得できませんでした」メッセージ

### BR-WEB-07: 未登録ペット
- クラウドにペットデータがない場合:
  - 「まだペットデータがありません。TUIでペットを育てて、セーブしてください」メッセージ

---

## 4. ダウンロードルール

### BR-WEB-08: OS検出
- navigator.userAgent からOS判定:
  - macOS: `Mac` を含む
  - Windows: `Win` を含む
  - Linux: `Linux` を含む
  - 不明: フォールバック（全OS分のコマンドを表示）

### BR-WEB-09: インストールコマンド表示

| OS | npm コマンド | 直接ダウンロード |
|----|------------|---------------|
| macOS | `npm install -g devpet` | GitHub Releases (darwin-amd64/arm64) |
| Windows | `npm install -g devpet` | GitHub Releases (windows-amd64) |
| Linux | `npm install -g devpet` | GitHub Releases (linux-amd64/arm64) |

---

## 5. セキュリティルール

### BR-WEB-10: HTTPセキュリティヘッダー（SECURITY-04準拠）
Next.jsのnext.config.jsで設定:
- Content-Security-Policy: `default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'`
  - `unsafe-inline` for styles: CSS-in-JSフレームワーク使用のため（文書化済み例外）
- Strict-Transport-Security: `max-age=31536000; includeSubDomains`
- X-Content-Type-Options: `nosniff`
- X-Frame-Options: `DENY`
- Referrer-Policy: `strict-origin-when-cross-origin`

### BR-WEB-11: CORS（SECURITY-08準拠）
- API呼び出しはバックエンドのCORS設定に依存
- Webサイトのドメインのみ許可

### BR-WEB-12: 入力バリデーション（SECURITY-05準拠）
- メールアドレス: RFC 5322準拠のフォーマットチェック
- パスワード: Cognitoポリシー準拠のクライアントサイドバリデーション
- デバイスコード: 英数字のみ、最大長制限
