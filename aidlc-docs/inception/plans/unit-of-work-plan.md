# ユニット生成計画

## 計画チェックリスト

### Phase 1: ユニット分解
- [x] アプリケーション設計のモジュール構成に基づきユニットを定義
- [x] 各ユニットの責務と成果物を明確化
- [x] ユニット間の依存関係を分析

### Phase 2: 実装順序の決定
- [x] 依存関係に基づく実装順序を決定
- [x] 並行開発可能なユニットを特定

### Phase 3: ユニットドキュメント生成
- [x] unit-of-work.md を生成（ユニット定義と責務）
- [x] unit-of-work-dependency.md を生成（依存関係マトリクス）
- [x] unit-of-work-story-map.md を生成（機能要件とユニットのマッピング）
- [x] コード構成戦略を文書化
- [x] ユニット境界と依存関係を検証

---

## 質問

以下の質問に回答してください。

## Question 1
ユニットの実装順序について、どの優先度で進めたいですか？

アプリケーション設計から、以下の5ユニットが特定されています：
1. **shared** — 共有Go module（型定義、バリデーション）
2. **backend** — AWSバックエンド（Lambda関数群）
3. **cli** — TUIクライアント
4. **web** — 紹介Webサイト（Next.js）
5. **cdk** — AWSインフラ（CDK）

A) バックエンド優先: shared → cdk → backend → cli → web（インフラとAPIを先に構築し、TUIとWebは後から接続）
B) TUI優先: shared → cli → cdk → backend → web（ローカル動作するTUIを先に作り、バックエンド接続は後から）
C) フルスタック順次: shared → cdk → backend → cli → web（インフラ→API→クライアント→Webの順）
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 2
コンストラクションフェーズでは、各ユニットごとに機能設計→NFR要件→NFR設計→インフラ設計→コード生成のループを回します。全ユニットを1つずつ完了させる方式と、設計段階をまとめて行う方式のどちらを好みますか？

A) ユニット単位で完了: shared完了 → backend完了 → cli完了 → web完了 → cdk完了（1ユニットずつ全工程を完了）
B) 段階まとめ: 全ユニットの機能設計 → 全ユニットのNFR → 全ユニットのコード生成（段階ごとにまとめて実行）
X) Other (please describe after [Answer]: tag below)

[Answer]: B
