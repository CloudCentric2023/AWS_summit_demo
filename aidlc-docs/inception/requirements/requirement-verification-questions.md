# devpet 要件確認質問

以下の質問に回答してください。各質問の `[Answer]:` タグの後に選択肢の文字を記入してください。
該当する選択肢がない場合は、最後の選択肢（Other）を選び、説明を記入してください。

---

## Question 1
TUIクライアントの実装言語・フレームワークはどれを想定していますか？

A) TypeScript + Ink（React for CLI）
B) TypeScript + blessed / blessed-contrib
C) Rust + ratatui
D) Go + bubbletea
E) Python + Textual / Rich
X) Other (please describe after [Answer]: tag below)

[Answer]: CとDで迷っている。
AWS SDKとの連携や、AIたまごっち風TUIツールを作ることを考慮した上で、二つの選択肢を比較検討して。

## Question 2
ペットの進化段階はどのくらいの数を想定していますか？

A) 3段階（たまご → こども → おとな）
B) 5段階（たまご → ベビー → こども → おとな → マスター）
C) 7段階以上（多段階進化、分岐進化あり）
X) Other (please describe after [Answer]: tag below)

[Answer]: A 後から増やすことを考慮した設計にして

## Question 3
ペットのパラメータについて、画像では「論理」「感情」「怠惰」が見えますが、これらのパラメータはペットの行動やAI応答にどう影響しますか？

A) 表示のみ（飾り）で、AI応答には影響しない
B) パラメータに応じてAIの応答トーンが変わる（論理が高い→論理的な返答、感情が高い→感情的な返答）
C) パラメータに応じてペットの進化先が分岐する
D) B + C の両方（応答トーンも変わり、進化先も分岐する）
X) Other (please describe after [Answer]: tag below)

[Answer]: あくまで「論理」「感情」「怠惰」は表示例である。（確定ではない）
パラメータによって応答トーンや進化先が変わるようにしたい。
その上で、色々と案を提示してほしい。

## Question 4
「話しかける」機能でのAI応答について、ユーザーは自由テキストで入力しますか？それとも選択肢ベースですか？

A) 自由テキスト入力（チャット形式）
B) 選択肢から選ぶ（定型文ベース）
C) 両方（自由テキスト + クイック選択肢）
X) Other (please describe after [Answer]: tag below)

[Answer]: X 「話しかける」ボタンを用意するだけで、ユーザーからの入力は受け付けない。

## Question 5
「24時間放置で死亡」について、死亡後の挙動はどうなりますか？

A) 完全リセット（新しいたまごからやり直し）
B) 墓石表示 + 新しいたまごで再スタート可能
C) 復活アイテムや課金で復活可能
D) 死亡はなく、HPが減るだけ（ペナルティのみ）
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 6
AWSバックエンドの構成について、どのサービスを想定していますか？

A) API Gateway + Lambda + DynamoDB（サーバーレス構成）
B) API Gateway + ECS/Fargate + RDS（コンテナ構成）
C) AppSync (GraphQL) + Lambda + DynamoDB
D) 特にこだわりなし、おすすめの構成で
X) Other (please describe after [Answer]: tag below)

[Answer]: ぼんやりと、ユーザーごとのデータ保持やLLM推論のためにAWSを使用したいと考えている。
どの構成を採用すると何が実現できるかを確認したい。
今回のケースで、データ保持以外にどのような機能が必用とかも知りたい。
改めて比較検討し提案して。

## Question 7
ユーザー認証はどのように行いますか？

A) 認証なし（匿名利用、デバイスIDベース）
B) Amazon Cognito（メール/パスワード）
C) ソーシャルログイン（GitHub, Google等）
D) APIキーベース（シンプルなトークン認証）
X) Other (please describe after [Answer]: tag below)

[Answer]: B TUI自体はユーザー登録をしなくても使用できるようにしたい。

## Question 8
紹介Webサイトのフレームワークはどれを想定していますか？

A) Next.js（React）
B) Astro（静的サイト生成）
C) Nuxt.js（Vue）
D) シンプルなHTML/CSS/JS（フレームワークなし）
X) Other (please describe after [Answer]: tag below)

[Answer]: A（React）

## Question 9
npmパッケージとしての配布について、パッケージ名やスコープの希望はありますか？

A) `devpet`（スコープなし）
B) `@devpet/cli`（スコープ付き）
C) まだ決めていない
X) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 10
ペットのデータ（レベル、パラメータ等）の保存先はどこですか？

A) ローカルのみ（~/.devpet/ などにJSON保存）
B) クラウドのみ（AWSバックエンドに保存）
C) ローカル + クラウド同期（オフラインでも動作、オンライン時に同期）
X) Other (please describe after [Answer]: tag below)

[Answer]: C 手動でsaveすることでローカルのデータをオンラインに保存する形にしたい。

## Question 11
「AI開発活動でレベルアップ」とありますが、具体的にどのような活動を検知しますか？

A) 「話しかける」ボタンを押すだけでEXP獲得（シンプルなクリッカー）
B) gitコミットやコード変更を検知してEXP獲得
C) A + B の両方
D) 時間経過でもEXP獲得（放置育成要素あり）
X) Other (please describe after [Answer]: tag below)

[Answer]: Aで考えている。
それに加えて、kiroやclaude codeのようなAIコーディングツールと連携して、hook機能を使うなどして、開発者のプロンプト送信を検知するとEXPを獲得できる機能も実装したい。
「話しかける」ボタンの方が、獲得EXPが多くなるようにして、開発者がボタンを押す導線を作りたい。

## Question 12
ターゲットユーザーの想定規模はどのくらいですか？（バックエンド設計に影響）

A) 個人利用・小規模（〜100人）
B) 中規模（100〜1,000人）
C) 大規模（1,000人以上）
D) まずは小規模で、スケール可能な設計にしたい
X) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question: セキュリティ拡張
このプロジェクトにセキュリティ拡張ルールを適用しますか？

A) はい — すべてのセキュリティルールをブロッキング制約として適用（本番グレードのアプリケーション向け推奨）
B) いいえ — セキュリティルールをスキップ（PoC、プロトタイプ、実験的プロジェクト向け）
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question: プロパティベーステスト拡張
このプロジェクトにプロパティベーステスト（PBT）ルールを適用しますか？

A) はい — すべてのPBTルールをブロッキング制約として適用（ビジネスロジック、データ変換、シリアライゼーション、ステートフルコンポーネントを持つプロジェクト向け推奨）
B) 部分的 — 純粋関数とシリアライゼーションのラウンドトリップにのみPBTルールを適用
C) いいえ — PBTルールをスキップ（シンプルなCRUDアプリ、UIのみのプロジェクト向け）
X) Other (please describe after [Answer]: tag below)

[Answer]: A
