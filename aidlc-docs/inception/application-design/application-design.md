# devpet アプリケーション設計（統合ドキュメント）

## 設計概要

devpetは以下の5つのモジュールで構成されるモノレポプロジェクト：

| モジュール | 言語 | 役割 |
|-----------|------|------|
| **shared** | Go | 共有型定義、バリデーション、定数 |
| **cli** | Go (bubbletea) | TUIクライアント |
| **backend** | Go | Lambda関数群（API） |
| **web** | TypeScript (Next.js) | 紹介Webサイト + 認証UI |
| **cdk** | TypeScript (CDK) | AWSインフラ定義 |

## 設計決定サマリー

| 決定事項 | 選択 | 理由 |
|---------|------|------|
| TUI画面構成 | メイン + 設定 + ヘルプ | シンプルな3画面構成 |
| APIエラー処理 | リトライなし、ローカルフォールバック | シンプルさ優先 |
| APIスタイル | リソースベース REST | REST標準、API Gatewayとの相性 |
| プロンプト管理 | Bedrock Prompt Management | バージョン管理、リアルタイムテスト、変数サポート |
| 認証連携 | デバイスコード認証フロー | UX最良、TUI→ブラウザ→自動反映 |
| 型共有 | 共有Go module | コンパイル時型安全性保証 |
| IaC配置 | モノレポ内 cdk/ | 一元管理 |

## REST API エンドポイント

| メソッド | パス | 説明 | 認証 |
|---------|------|------|------|
| POST | `/pets/{petId}/talk` | 話しかける（AI応答生成） | 必須 |
| PUT | `/pets/{petId}/save` | クラウドセーブ | 必須 |
| GET | `/pets/{petId}` | クラウドロード | 必須 |
| POST | `/auth/device` | デバイスコード認証開始 | 不要 |
| POST | `/auth/device/poll` | デバイス認証ポーリング | 不要 |

## 認証フロー（デバイスコード認証）

```
TUI                    Backend              Web                  Cognito
 |                        |                   |                      |
 |-- POST /auth/device -->|                   |                      |
 |<-- device_code, url ---|                   |                      |
 |                        |                   |                      |
 |-- open browser --------|------------------>|                      |
 |                        |                   |-- login/signup ----->|
 |                        |                   |<-- tokens -----------|
 |                        |                   |-- approve device --->|
 |                        |                   |                      |
 |-- POST /auth/poll ---->|                   |                      |
 |<-- access_token -------|                   |                      |
 |                        |                   |                      |
 | (save token locally)   |                   |                      |
```

## データフロー概要

### 話しかけフロー
1. ユーザーが `[t]` キーを押す
2. TUI Core (PetEngine) がEXP計算・パラメータ変動を適用
3. TUI Net (APIClient) がバックエンドに POST `/pets/{petId}/talk`
4. Lambda (TalkHandler) → AIService → Bedrock Prompt Mgmt → Bedrock (Claude Haiku)
5. AI応答テキストをTUIに返却
6. TUI UI (SpeechBubble) に応答を表示
7. エラー時: ローカルモードにフォールバック（定型文を表示）

### セーブフロー
1. ユーザーが `[s]` キーを押す
2. TUI Core がローカルデータを保存
3. 認証済みの場合: TUI Net がバックエンドに PUT `/pets/{petId}/save`
4. Lambda (SaveHandler) → PetService → DynamoDB に保存
5. 未認証の場合: ローカル保存のみ

### 起動フロー
1. TUI起動
2. LocalStorage からペットデータを読み込み
3. TimeCalculator で経過時間を計算
4. パラメータ自然変動を適用（きずな低下、きぶん変動、たいりょく減少等）
5. 死亡判定（たいりょく=0なら死亡画面へ）
6. メイン画面を表示

## 詳細設計ドキュメント

- [components.md](components.md) — コンポーネント定義と責務
- [component-methods.md](component-methods.md) — メソッドシグネチャ
- [services.md](services.md) — サービス定義とオーケストレーション
- [component-dependency.md](component-dependency.md) — 依存関係とデータフロー
