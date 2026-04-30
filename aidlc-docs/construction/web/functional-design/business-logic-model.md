# web — ビジネスロジックモデル

## 概要
紹介Webサイトはビジネスロジックが軽量。主な責務は:
1. devpetの紹介・ダウンロード案内
2. Cognito認証UI（サインアップ/サインイン）
3. デバイスコード認証の承認フロー
4. ログイン後のダッシュボード（最小限の情報表示）

---

## 1. ページ構成

### LandingPage（トップページ）
```
責務: devpetの紹介、ダウンロード案内
認証: 不要

表示内容:
- ヒーローセクション（キャッチコピー + ペットAA表示）
- 機能紹介セクション（3〜4つの特徴）
- ダウンロードセクション（OS検出 + インストールコマンド）
- フッター

デザインテイスト: ゆるい・かわいい系（パステルカラー、丸みのあるデザイン）
```

### AuthPage（認証ページ）
```
責務: サインアップ/サインイン
認証: 不要（認証フロー自体）

表示内容:
- サインアップフォーム（メール、パスワード）
- サインインフォーム（メール、パスワード）
- タブ切り替え（サインアップ ↔ サインイン）
- メール確認コード入力（サインアップ後）

Cognito連携: Amplify Auth SDK
```

### DashboardPage（ダッシュボード）
```
責務: ログイン後のアカウント管理
認証: 必須

表示内容（最小限）:
- 現在のペット名
- 現在のレベル
- 現在の進化段階
- ログアウトボタン

データ取得: バックエンドAPI (GET /pets/{petId}) から取得
```

### DeviceAuthPage（デバイス認証ページ）
```
責務: TUIからのデバイスコード認証フローを処理
認証: 必須（ログイン済みユーザーがデバイスを承認）

フロー:
1. TUIがブラウザでこのページを開く（URLにuser_codeを含む）
2. ユーザーがログイン済みでない場合 → AuthPageにリダイレクト
3. ログイン済みの場合 → デバイスコードの確認画面を表示
4. ユーザーが「承認」ボタンを押す
5. バックエンドAPIにデバイス承認リクエストを送信
6. 承認完了メッセージを表示
7. TUI側でポーリングにより認証完了を検知
```

---

## 2. コンポーネント設計

### AuthProvider
```
責務: アプリ全体の認証状態管理
実装: React Context + Amplify Auth SDK

状態:
- isAuthenticated: boolean
- user: CognitoUser | null
- isLoading: boolean

メソッド:
- signUp(email, password): Promise<void>
- confirmSignUp(email, code): Promise<void>
- signIn(email, password): Promise<void>
- signOut(): Promise<void>
- getAccessToken(): Promise<string>
```

### DownloadSection
```
責務: OS検出とインストールコマンド表示

ロジック:
1. navigator.userAgent からOSを検出
2. OS に応じたインストールコマンドを表示:
   - macOS/Linux: `npm install -g devpet` or `curl -fsSL https://...`
   - Windows: `npm install -g devpet` or PowerShellスクリプト
3. GitHub Releasesへのリンクも表示
```

---

## 3. デザインルール

### カラーパレット（ゆるい・かわいい系）

| 要素 | カラー | 用途 |
|------|--------|------|
| Primary | `#7C3AED` (紫) | ボタン、リンク、アクセント |
| Secondary | `#EC4899` (ピンク) | サブアクセント |
| Background | `#FFF7ED` (クリーム) | ページ背景 |
| Surface | `#FFFFFF` (白) | カード背景 |
| Text | `#1F2937` (ダークグレー) | 本文テキスト |
| Muted | `#9CA3AF` (グレー) | 補助テキスト |

### タイポグラフィ
- 見出し: 丸ゴシック系（例: Rounded Mplus 1c）
- 本文: ゴシック系（例: Noto Sans JP）
- コードブロック: 等幅フォント

### レイアウト
- 最大幅: 1200px（中央寄せ）
- 角丸: 12px〜16px
- シャドウ: 柔らかいドロップシャドウ

---

## テスト可能プロパティ（PBT-01）

| コンポーネント | プロパティカテゴリ | プロパティ説明 |
|--------------|-----------------|-------------|
| DownloadSection | Invariant | 任意のuserAgentに対して常にダウンロードコマンドを表示（未知のOSでもフォールバック） |
| AuthProvider | Invariant | signOut後はisAuthenticated=false |
| AuthProvider | Invariant | signIn成功後はisAuthenticated=true |

> **Note**: WebフロントエンドのPBTは限定的。主にユーティリティ関数やデータ変換に適用。UIコンポーネントは主にexample-basedテストでカバー。
