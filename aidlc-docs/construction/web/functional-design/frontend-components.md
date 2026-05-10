# web — フロントエンドコンポーネント設計

## 1. ページコンポーネント

### LandingPage (`/`)

```typescript
// Props: なし
// 状態: なし（静的ページ）

構成:
├── Header（ナビゲーション）
│   ├── ロゴ（devpet）
│   ├── ナビリンク（Features, Download, Login）
│   └── CTAボタン（Get Started）
├── HeroSection
│   ├── キャッチコピー
│   ├── サブテキスト
│   └── ペットAAイメージ（静的画像 or CSS art）
├── FeaturesSection
│   ├── FeatureCard × 3〜4
│   │   ├── アイコン
│   │   ├── タイトル
│   │   └── 説明文
├── DownloadSection
│   ├── OS検出結果
│   ├── インストールコマンド（コピーボタン付き）
│   └── GitHub Releasesリンク
└── Footer
    ├── GitHubリンク
    └── コピーライト
```

### AuthPage (`/auth`)

```typescript
// Props: なし
// 状態:
//   - mode: 'signin' | 'signup' | 'confirm'
//   - email: string
//   - password: string
//   - confirmCode: string
//   - error: string | null
//   - isLoading: boolean

構成:
├── AuthCard
│   ├── タブ切り替え（サインイン / サインアップ）
│   ├── SignInForm
│   │   ├── メール入力
│   │   ├── パスワード入力
│   │   └── サインインボタン
│   ├── SignUpForm
│   │   ├── メール入力
│   │   ├── パスワード入力
│   │   ├── パスワード確認入力
│   │   └── サインアップボタン
│   └── ConfirmForm（サインアップ後）
│       ├── 確認コード入力
│       └── 確認ボタン
└── エラーメッセージ表示
```

### DashboardPage (`/dashboard`)

```typescript
// Props: なし
// 状態:
//   - pet: Pet | null
//   - isLoading: boolean
//   - error: string | null
// 認証: 必須（未認証時は /auth にリダイレクト）

構成:
├── DashboardHeader
│   ├── 「ようこそ、{email}さん」
│   └── ログアウトボタン
├── PetInfoCard（ペットデータがある場合）
│   ├── ペット名
│   ├── レベル
│   └── 進化段階
├── NoPetMessage（ペットデータがない場合）
│   └── 「TUIでペットを育てて、セーブしてください」
└── Footer
```

### DeviceAuthPage (`/auth/device`)

```typescript
// Props: なし
// 状態:
//   - userCode: string（URLパラメータから取得）
//   - status: 'confirming' | 'approving' | 'success' | 'error'
//   - error: string | null
// 認証: 必須（未認証時は /auth にリダイレクト、リダイレクト先を保持）

構成:
├── DeviceAuthCard
│   ├── 説明テキスト（「TUIからの認証リクエスト」）
│   ├── デバイスコード表示
│   ├── 承認ボタン
│   └── キャンセルボタン
├── SuccessMessage（承認成功時）
│   └── 「認証が完了しました。このページを閉じてください」
└── ErrorMessage（エラー時）
    └── エラー内容表示
```

---

## 2. 共通コンポーネント

### AuthProvider

```typescript
// React Context Provider
// Amplify Auth SDKをラップ

interface AuthContextType {
  isAuthenticated: boolean;
  user: CognitoUser | null;
  isLoading: boolean;
  signUp: (email: string, password: string) => Promise<void>;
  confirmSignUp: (email: string, code: string) => Promise<void>;
  signIn: (email: string, password: string) => Promise<void>;
  signOut: () => Promise<void>;
  getAccessToken: () => Promise<string>;
}
```

### DownloadSection

```typescript
// Props: なし
// 状態:
//   - detectedOS: 'macos' | 'windows' | 'linux' | 'unknown'
//   - copied: boolean（コピーボタンのフィードバック）

// OS検出ロジック
function detectOS(): 'macos' | 'windows' | 'linux' | 'unknown' {
  const ua = navigator.userAgent;
  if (ua.includes('Mac')) return 'macos';
  if (ua.includes('Win')) return 'windows';
  if (ua.includes('Linux')) return 'linux';
  return 'unknown';
}
```

---

## 3. ルーティング

| パス | ページ | 認証 |
|------|--------|------|
| `/` | LandingPage | 不要 |
| `/auth` | AuthPage | 不要 |
| `/dashboard` | DashboardPage | 必須 |
| `/auth/device` | DeviceAuthPage | 必須 |

### 認証ガード
- 認証必須ページにアクセス時、未認証なら `/auth` にリダイレクト
- リダイレクト元のパスをクエリパラメータで保持（`/auth?redirect=/dashboard`）
- 認証成功後にリダイレクト元に遷移

---

## 4. API通信

```typescript
// バックエンドAPI呼び出し用クライアント
class ApiClient {
  private baseUrl: string;
  
  // ペットデータ取得（ダッシュボード用）
  async getPet(accessToken: string): Promise<Pet | null>;
  
  // デバイス認証承認
  async approveDevice(accessToken: string, userCode: string): Promise<void>;
}
```

---

## 5. アクセシビリティ

### WCAG 2.1 AA準拠目標
- カラーコントラスト比: 4.5:1以上（通常テキスト）、3:1以上（大きいテキスト）
- キーボードナビゲーション: 全操作がキーボードで可能
- スクリーンリーダー: 適切なaria-label、role属性
- フォーカス管理: 可視的なフォーカスインジケーター
- フォームラベル: 全入力フィールドにラベル関連付け
