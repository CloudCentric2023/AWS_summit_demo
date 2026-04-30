# 機能設計計画（全ユニット — 段階まとめ方式）

## 概要
段階まとめ方式に従い、全5ユニット（shared, cli, cdk, backend, web）の機能設計をまとめて実施する。

## 実行計画

### Phase 1: 質問収集・回答分析
- [x] Step 1: 全ユニットの機能設計に必要な質問を作成
- [x] Step 2: ユーザー回答を収集・分析
- [x] Step 3: 矛盾・曖昧さの解消（矛盾なし。提案項目は設計ドキュメント内で対応）

### Phase 2: Unit 1 — shared 機能設計
- [x] Step 4: ドメインエンティティ定義（Pet, TalkRequest/Response, SaveRequest/Response等）
- [x] Step 5: バリデーションルール定義（パラメータ範囲、データ整合性）
- [x] Step 6: 定数定義（進化段階、パラメータ上限、EXP係数等）
- [x] Step 7: テスト可能プロパティの特定（PBT-01）

### Phase 3: Unit 2 — cli 機能設計
- [x] Step 8: ビジネスロジックモデル（PetEngine, TimeCalculator, ExpCalculator, EvolutionManager）
- [x] Step 9: パラメータ計算ロジック詳細（自然変動、話しかけ変動、相互作用）
- [x] Step 10: 進化判定ロジック（パラメータ傾向→進化先分岐）
- [x] Step 11: 死亡・復活ロジック
- [x] Step 12: EXP・レベルアップ計算ロジック
- [x] Step 13: フロントエンドコンポーネント設計（TUI画面構成、状態管理）
- [x] Step 14: テスト可能プロパティの特定（PBT-01）

### Phase 4: Unit 3 — cdk 機能設計
- [x] Step 15: CDKスタック構成のビジネスルール（リソース命名、タグ付け）
- [x] Step 16: テスト可能プロパティの特定（PBT-01）

### Phase 5: Unit 4 — backend 機能設計
- [x] Step 17: API処理ロジック（TalkHandler, SaveHandler, LoadHandler, AuthHandler）
- [x] Step 18: AIService ビジネスロジック（プロンプト構築、応答生成）
- [x] Step 19: PetService ビジネスロジック（CRUD、バリデーション）
- [x] Step 20: テスト可能プロパティの特定（PBT-01）

### Phase 6: Unit 5 — web 機能設計
- [x] Step 21: フロントエンドコンポーネント設計（ページ構成、認証フロー）
- [x] Step 22: テスト可能プロパティの特定（PBT-01）

### Phase 7: 成果物生成・レビュー
- [x] Step 23: 全ユニットの機能設計ドキュメント生成
- [ ] Step 24: ユーザー承認
