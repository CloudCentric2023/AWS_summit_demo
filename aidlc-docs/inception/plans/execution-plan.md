# 実行計画

## 詳細分析サマリー

### 変更影響評価
- **ユーザー向け変更**: あり — TUIペット育成体験、Webサイトでの登録/ログイン
- **構造変更**: あり — 3コンポーネント（TUI + Web + AWS）の新規構築
- **データモデル変更**: あり — DynamoDBテーブル設計（Users, Pets, Graveyard）
- **API変更**: あり — REST API新規設計（AI応答、データ管理）
- **NFR影響**: あり — セキュリティ（SECURITY-01〜15）、PBT（PBT-01〜10）

### リスク評価
- **リスクレベル**: Medium
- **ロールバック複雑さ**: 低（グリーンフィールドのため）
- **テスト複雑さ**: 中（3コンポーネント間の結合テストが必要）

---

## ワークフロー可視化

```mermaid
flowchart TD
    Start(["ユーザーリクエスト"])

    subgraph INCEPTION["🔵 インセプションフェーズ"]
        WD["ワークスペース検出<br/><b>COMPLETED</b>"]
        RA["要件分析<br/><b>COMPLETED</b>"]
        US["ユーザーストーリー<br/><b>SKIP</b>"]
        WP["ワークフロー計画<br/><b>IN PROGRESS</b>"]
        AD["アプリケーション設計<br/><b>EXECUTE</b>"]
        UG["ユニット生成<br/><b>EXECUTE</b>"]
    end

    subgraph CONSTRUCTION["🟢 コンストラクションフェーズ"]
        FD["機能設計<br/><b>EXECUTE</b>"]
        NFRA["NFR要件<br/><b>EXECUTE</b>"]
        NFRD["NFR設計<br/><b>EXECUTE</b>"]
        ID["インフラ設計<br/><b>EXECUTE</b>"]
        CG["コード生成<br/><b>EXECUTE</b>"]
        BT["ビルド & テスト<br/><b>EXECUTE</b>"]
    end

    Start --> WD
    WD --> RA
    RA --> WP
    WP --> AD
    AD --> UG
    UG --> FD
    FD --> NFRA
    NFRA --> NFRD
    NFRD --> ID
    ID --> CG
    CG --> BT
    BT --> End(["完了"])

    style WD fill:#4CAF50,stroke:#1B5E20,stroke-width:3px,color:#fff
    style RA fill:#4CAF50,stroke:#1B5E20,stroke-width:3px,color:#fff
    style US fill:#BDBDBD,stroke:#424242,stroke-width:2px,stroke-dasharray: 5 5,color:#000
    style WP fill:#4CAF50,stroke:#1B5E20,stroke-width:3px,color:#fff
    style AD fill:#FFA726,stroke:#E65100,stroke-width:3px,stroke-dasharray: 5 5,color:#000
    style UG fill:#FFA726,stroke:#E65100,stroke-width:3px,stroke-dasharray: 5 5,color:#000
    style FD fill:#FFA726,stroke:#E65100,stroke-width:3px,stroke-dasharray: 5 5,color:#000
    style NFRA fill:#FFA726,stroke:#E65100,stroke-width:3px,stroke-dasharray: 5 5,color:#000
    style NFRD fill:#FFA726,stroke:#E65100,stroke-width:3px,stroke-dasharray: 5 5,color:#000
    style ID fill:#FFA726,stroke:#E65100,stroke-width:3px,stroke-dasharray: 5 5,color:#000
    style CG fill:#4CAF50,stroke:#1B5E20,stroke-width:3px,color:#fff
    style BT fill:#4CAF50,stroke:#1B5E20,stroke-width:3px,color:#fff
    style Start fill:#CE93D8,stroke:#6A1B9A,stroke-width:3px,color:#000
    style End fill:#CE93D8,stroke:#6A1B9A,stroke-width:3px,color:#000
    style INCEPTION fill:#BBDEFB,stroke:#1565C0,stroke-width:3px,color:#000
    style CONSTRUCTION fill:#C8E6C9,stroke:#2E7D32,stroke-width:3px,color:#000

    linkStyle default stroke:#333,stroke-width:2px
```

### テキスト代替（ワークフロー可視化）
```
インセプションフェーズ:
  ワークスペース検出 (COMPLETED) → 要件分析 (COMPLETED) → ワークフロー計画 (IN PROGRESS)
  → アプリケーション設計 (EXECUTE) → ユニット生成 (EXECUTE)
  ※ ユーザーストーリー (SKIP)

コンストラクションフェーズ（ユニットごと）:
  機能設計 (EXECUTE) → NFR要件 (EXECUTE) → NFR設計 (EXECUTE)
  → インフラ設計 (EXECUTE) → コード生成 (EXECUTE) → ビルド&テスト (EXECUTE)
```

---

## 実行するフェーズ

### 🔵 インセプションフェーズ
- [x] ワークスペース検出 (COMPLETED)
- [x] 要件分析 (COMPLETED)
- [x] ワークフロー計画 (IN PROGRESS)
- [ ] アプリケーション設計 - **EXECUTE**
  - **理由**: 3コンポーネント（TUI/Web/AWS）の新規設計が必要。コンポーネント間のAPI設計、サービスレイヤー定義、依存関係の明確化が必要
- [ ] ユニット生成 - **EXECUTE**
  - **理由**: 3つの独立したコンポーネント（TUIクライアント、紹介Webサイト、AWSバックエンド）に分解し、それぞれの実装順序と依存関係を定義する必要がある

### 🟢 コンストラクションフェーズ（ユニットごと）
- [ ] 機能設計 - **EXECUTE**
  - **理由**: パラメータ計算ロジック、進化判定、EXP計算、死亡判定、AI応答生成のビジネスロジックが複雑で詳細設計が必要
- [ ] NFR要件 - **EXECUTE**
  - **理由**: セキュリティ拡張（SECURITY-01〜15）とPBT拡張（PBT-01〜10）が有効。技術スタック選定の確認も必要
- [ ] NFR設計 - **EXECUTE**
  - **理由**: NFR要件で特定されたパターンをアーキテクチャに組み込む設計が必要
- [ ] インフラ設計 - **EXECUTE**
  - **理由**: API Gateway、Lambda、DynamoDB、Cognito、Bedrockの具体的なリソース設計とCDKテンプレート設計が必要
- [ ] コード生成 - **EXECUTE**（常時実行）
  - **理由**: 実装計画の策定とコード生成
- [ ] ビルド & テスト - **EXECUTE**（常時実行）
  - **理由**: ビルド手順、テスト実行手順の策定

## スキップするフェーズ

### 🔵 インセプションフェーズ
- ユーザーストーリー - **SKIP**
  - **理由**: 要件分析で機能要件（FR-01〜FR-11）が十分に定義済み。ユーザータイプは開発者のみで単一ペルソナ。ユーザーストーリーの追加価値が限定的

### 🟡 オペレーションフェーズ
- オペレーション - **PLACEHOLDER**
  - **理由**: 将来の拡張用プレースホルダー

---

## 成功基準
- **主目標**: devpet TUIツールが動作し、ペット育成・AI応答・クラウド同期が機能すること
- **主要成果物**:
  - Go + bubbletea TUIクライアント（クロスプラットフォームバイナリ）
  - Next.js紹介Webサイト（Cognito登録/ログイン付き）
  - AWSバックエンド（API Gateway + Lambda + DynamoDB + Bedrock + Cognito）
  - AWS CDK IaCテンプレート
  - PBTを含む包括的テストスイート
- **品質ゲート**:
  - セキュリティ拡張ルール（SECURITY-01〜15）準拠
  - PBT拡張ルール（PBT-01〜10）準拠
  - クロスプラットフォームビルド成功（Linux, macOS, Windows）
