# devpet ユニット依存関係

## 依存関係マトリクス

```
依存元 ＼ 依存先  | shared | cli | cdk | backend | web |
------------------|--------|-----|-----|---------|-----|
shared            |   -    |     |     |         |     |
cli               |   ●    |  -  |     |         |     |
cdk               |        |     |  -  |         |     |
backend           |   ●    |     |     |    -    |     |
web               |        |     |     |         |  -  |

● = ビルド時依存（Go module import）
```

## 依存関係図

```
                shared
               /      \
              v        v
            cli      backend
              \      /
               \    /
                v  v
          (API通信: ランタイム依存)
                |
                v
               cdk  ←--- backend (デプロイ対象)
                |
                v
               web  ←--- cdk (ホスティング対象)
```

## 依存関係の詳細

### ビルド時依存（コンパイル時）

| 依存元 | 依存先 | 依存内容 |
|--------|--------|---------|
| cli → shared | Go module import | Pet構造体、TalkRequest/Response、バリデーション関数、定数 |
| backend → shared | Go module import | Pet構造体、TalkRequest/Response、バリデーション関数、定数 |

### デプロイ時依存

| 依存元 | 依存先 | 依存内容 |
|--------|--------|---------|
| cdk → backend | Lambda関数バイナリ | CDKがbackendのGoバイナリをLambda関数としてデプロイ |
| cdk → web | Next.jsビルド成果物 | CDKがwebのビルド成果物をS3/CloudFrontにデプロイ |

### ランタイム依存

| 依存元 | 依存先 | 依存内容 |
|--------|--------|---------|
| cli → backend | HTTPS REST API | 話しかけ、セーブ、ロード、認証 |
| cli → web | ブラウザ起動 | デバイスコード認証フローでWebサイトを開く |
| web → backend | HTTPS REST API | デバイス認証承認 |

### 外部サービス依存

| ユニット | 外部サービス | 用途 |
|---------|-------------|------|
| backend | Amazon Bedrock (Claude Haiku) | AI応答生成 |
| backend | Bedrock Prompt Management | プロンプトテンプレート管理 |
| backend | Amazon DynamoDB | ペット/ユーザーデータ永続化 |
| backend | Amazon Cognito | トークン検証 |
| cli | Amazon Cognito | デバイスコード認証 |
| web | Amazon Cognito | サインアップ/サインイン |

## 実装順序と依存関係の整合性

```
Step 1: shared    ← 依存なし。最初に実装可能
Step 2: cli       ← shared に依存。shared完了後に実装可能。ローカルモードで単独動作
Step 3: cdk       ← 依存なし（IaC定義のみ）。backend/webのビルド成果物は後から接続
Step 4: backend   ← shared に依存。cdk完了後にデプロイ可能
Step 5: web       ← cdk完了後にデプロイ可能
```

## 並行開発の可能性

| 並行可能な組み合わせ | 条件 |
|-------------------|------|
| cli + cdk | shared完了後、cliとcdkは独立して開発可能 |
| backend + web | cdk完了後、backendとwebは独立して開発可能 |
