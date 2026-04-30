# アプリケーション設計計画

## 設計計画チェックリスト

### Phase 1: コンポーネント設計
- [x] TUIクライアントのコンポーネント構成を定義
- [x] AWSバックエンドのコンポーネント構成を定義
- [x] 紹介Webサイトのコンポーネント構成を定義

### Phase 2: コンポーネントメソッド定義
- [x] TUIクライアントのメソッドシグネチャを定義
- [x] AWSバックエンド（Lambda関数）のメソッドシグネチャを定義
- [x] 紹介WebサイトのAPIルート/コンポーネントを定義

### Phase 3: サービスレイヤー設計
- [x] API設計（REST エンドポイント定義）
- [x] サービス間通信パターンの定義
- [x] 認証フローの設計

### Phase 4: コンポーネント依存関係
- [x] コンポーネント間の依存関係マトリクスを作成
- [x] データフロー図を作成
- [x] 通信パターンを定義

### Phase 5: 設計ドキュメント生成
- [x] components.md を生成
- [x] component-methods.md を生成
- [x] services.md を生成
- [x] component-dependency.md を生成
- [x] application-design.md（統合ドキュメント）を生成
- [x] 設計の完全性と一貫性を検証

---

## 設計質問

以下の質問に回答してください。各質問の `[Answer]:` タグの後に選択肢の文字を記入してください。

### コンポーネント設計

## Question 1
TUIクライアントの画面構成について、メイン画面以外にどのような画面が必要ですか？

A) メイン画面（ペット表示）のみ。設定やセーブはキーバインドで操作
B) メイン画面 + 設定画面（認証設定、表示設定等）
C) メイン画面 + 設定画面 + 墓地画面（過去のペット一覧）
D) メイン画面 + 設定画面 + 墓地画面 + ヘルプ画面
X) Other (please describe after [Answer]: tag below)

[Answer]: メイン画面 + 設定画面 + ヘルプ画面

## Question 2
TUIクライアントからAWSバックエンドへのAPI呼び出しについて、エラー時のリトライ戦略はどうしますか？

A) リトライなし（エラー表示してローカルモードにフォールバック）
B) 最大3回リトライ（指数バックオフ）、失敗時はローカルモードにフォールバック
C) バックグラウンドでリトライキューに入れ、後で再送
X) Other (please describe after [Answer]: tag below)

[Answer]: A

### サービスレイヤー設計

## Question 3
REST APIのエンドポイント設計について、以下のどのスタイルを好みますか？

A) リソースベース（`/pets/{petId}/talk`, `/pets/{petId}/save`, `/users/register`）
B) アクションベース（`/api/talk`, `/api/save`, `/api/register`）
C) バージョン付きリソースベース（`/v1/pets/{petId}/talk`, `/v1/pets/{petId}/save`）
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 4
AI応答生成のプロンプト設計について、ペットの「キャラクター設定」はどこで管理しますか？

A) バックエンド側（Lambda内にプロンプトテンプレートを保持）
B) DynamoDBに保存（進化段階ごとのキャラクター設定をDBで管理、管理画面なしで直接DB編集）
C) 設定ファイル（S3やParameter Storeに外部化）
X) Other (please describe after [Answer]: tag below)

[Answer]: Bedrock Prompt Managementを使用して

## Question 5
Webサイトでのログイン後、TUIクライアントとの認証トークン連携はどのように行いますか？

A) Webサイトでログイン後、認証トークンを表示 → ユーザーがTUIに手動入力（`devpet login --token xxx`）
B) Webサイトでログイン後、デバイスコード認証フロー（TUIで `devpet login` → ブラウザが開く → Web側で認証 → TUIに自動反映）
C) Webサイトでログイン後、認証情報をファイルに保存 → TUIが同じファイルを読み取り
X) Other (please describe after [Answer]: tag below)

[Answer]: B

### コンポーネント依存関係

## Question 6
TUIクライアントとAWSバックエンドの間で共有する型定義（ペットデータ構造等）はどう管理しますか？

A) 各コンポーネントで独立して定義（API仕様書で整合性を担保）
B) 共有パッケージ（Go module）として切り出し、TUIとLambdaで共有
C) OpenAPI仕様から自動生成
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 7
CDK（IaC）のコードはどこに配置しますか？

A) 同一リポジトリ内の `infra/` ディレクトリ（モノレポ）
B) 別リポジトリ
C) 同一リポジトリ内の `cdk/` ディレクトリ（モノレポ）
X) Other (please describe after [Answer]: tag below)

[Answer]: C
