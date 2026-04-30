# cli — ビジネスロジックモデル

## 1. PetEngine（ペットエンジン）

ペットの全状態管理を統括するコアコンポーネント。

### Init フロー
```
1. LocalStorage.Load() でペットデータを読み込み
2. データが存在しない場合 → 新規ペット生成（初期値で作成）
3. データが存在する場合:
   a. TimeCalculator.CalcDeath() で死亡判定
   b. 死亡の場合 → IsDead=true, DiedAt設定, DeathScreen表示
   c. 生存の場合 → TimeCalculator.CalcElapsed() でパラメータ自然変動を適用
   d. ExpCalculator.CalcLevelUp() でレベルアップ判定
   e. EvolutionManager.CheckEvolution() で進化判定
4. LocalStorage.Save() で更新データを保存
5. Pet を返却
```

### Talk フロー
```
1. ペットが死亡中の場合 → エラー返却
2. ExpCalculator.CalcTalkExp() でEXP獲得量を計算
3. Pet.Exp += 獲得EXP
4. パラメータ変動を適用:
   - きずな: +3〜+5（ランダム）
   - かしこさ: +1〜+2（ランダム）
   - きぶん: -2〜+3（ランダム）
   - HP: -1〜-3（ランダム、連打ペナルティ）
5. ClampParam() で全パラメータをクランプ
6. ExpCalculator.CalcLevelUp() でレベルアップ判定
7. EvolutionManager.CheckEvolution() で進化判定
8. CheckDeath() で死亡判定（HP=0チェック）
9. オンライン + 認証済みの場合:
   a. APIClient.Talk() でバックエンドにAI応答をリクエスト
   b. 成功 → AI応答テキストを返却
   c. 失敗 → ローカルフォールバック（定型文）
10. オフライン or 未認証の場合:
    → ローカルフォールバック（定型文）
11. LastInteractionAt を現在時刻に更新
12. LocalStorage.Save() で保存
13. TalkResult を返却
```

### Tick フロー（1秒ごとの定期処理）
```
1. 前回Tickからの経過秒数を計算
2. パラメータ微小変動を適用（1秒あたりの変動量）:
   - きぶん: 時間帯に応じた自然変動（後述）
   - HP: 微小回復（+0.01/秒 = +36/時間、ただし上限100）
3. ClampParam() で全パラメータをクランプ
4. 死亡判定（HP=0チェック）
5. 画面更新をトリガー
```

---

## 2. TimeCalculator（経過時間計算）

TUI起動時に前回操作時刻からの経過時間を計算し、パラメータ変動をまとめて適用する。

### CalcDeath
```
入力: Pet, 現在時刻
出力: bool (死亡かどうか)

ルール:
- 経過時間 = 現在時刻 - Pet.LastInteractionAt
- 経過時間 ≥ 24時間 → true（死亡）
- それ以外 → false
```

### CalcElapsed
```
入力: Pet, 現在時刻
出力: ParamDelta

経過時間 = 現在時刻 - Pet.LastInteractionAt（時間単位）

パラメータ変動計算:
- HP: -3/時間（放置による体力低下）
  → 24時間で -72。HP100から開始すると約33時間で0に到達
  → ただし24時間ルールで先に死亡するため、実質的にはHP28まで低下
- きずな: -2/時間（放置による絆低下）
  → 24時間で -48
- かしこさ: -0.5/時間（放置による微減）
  → 24時間で -12
- きぶん: -1/時間（放置による気分低下）
  → 24時間で -24

全変動量にClampParamを適用。
```

**パラメータ変動速度の設計根拠:**
- 24時間死亡ルールとのバランス: HPは24時間で0にならない速度（-3/h）だが、24時間放置で死亡判定が先に発動
- きずなは最も減りやすい（-2/h）: 「話しかけないと疎遠になる」感覚を演出
- かしこさは最も減りにくい（-0.5/h）: 「知識は簡単には失われない」
- きぶんは中程度（-1/h）: 自然な気分の低下

---

## 3. ExpCalculator（EXP計算）

### CalcTalkExp
```
入力: Pet
出力: int (獲得EXP)

基本値: 100 EXP（固定）
```

### CalcHookExp
```
入力: Pet
出力: int (獲得EXP)

基本値: 10 EXP（固定）
```

### CalcLevelUp
```
入力: Pet
出力: (newLevel int, leveledUp bool)

1. 現在のレベルを CalcLevel(Pet.Exp) で算出
2. newLevel > Pet.Level の場合 → leveledUp = true
3. Pet.Level = newLevel に更新
```

---

## 4. EvolutionManager（進化管理）

### CheckEvolution
```
入力: Pet
出力: (*EvolutionResult, error)

1. 現在の Stage と Level を確認
2. 進化条件チェック:
   - Stage=egg かつ Level ≥ 10 → こどもに進化
   - Stage=child かつ Level ≥ 20 → おとなに進化
   - それ以外 → 進化なし

3. こどもへの進化（egg → child）:
   a. かしこさ, きずな, きぶん の3パラメータを比較（HPは進化判定に使用しない）
   b. 最も高いパラメータに対応する進化タイプを選択
      - かしこさ最高 → brain
      - きずな最高 → heart
      - きぶん最高 → mood
   c. 同率タイブレーク: きずな > かしこさ > きぶん

4. おとなへの進化（child → adult）:
   a. 育成失敗チェック（最優先）:
      かしこさ ≤ 30 かつ きずな ≤ 30 かつ きぶん ≤ 30 → stray
   b. こども段階の EvolutionType に応じた分岐:
      - brain → きずな ≥ きぶん なら sage、そうでなければ scholar
      - heart → かしこさ ≥ きぶん なら soulmate、そうでなければ cheerful
      - mood  → かしこさ ≥ きずな なら trickster、そうでなければ free-spirit

5. Pet.Stage を更新
6. Pet.EvolutionType を設定
7. EvolutionResult を返却（進化したかどうか、新しいStage、EvolutionType）
```

### GetEvolutionCandidates
```
入力: Pet
出力: []EvolutionCandidate

現在のパラメータ値に基づいて、到達可能な進化先一覧を返す。
（UI表示用: 「あと○○ポイントで△△タイプに進化」等）
```

### GetAsciiArt
```
入力: stage (Stage), evolutionType (string)
出力: string (AAアート)

進化段階と進化タイプに応じたAAアートを返す。
```

---

## 5. パラメータ相互作用ルール

要件定義書 FR-03 で定義されたパラメータ相互作用をAI応答プロンプトに反映する。

| 条件 | AI応答トーン |
|------|------------|
| きずな ≥ 70 かつ かしこさ ≥ 70 | 「頼れる相棒」感の返答 |
| きぶん ≤ 30 かつ きずな ≥ 70 | 「だるいけど…あなただから話すね」的な返答 |
| HP ≤ 30 | きぶんも連動して低下しやすい（Tick処理で反映） |
| かしこさ ≥ 70 かつ きぶん ≤ 30 | 「知ってるけど教えない」的なツンデレ返答 |

**HP→きぶん連動ルール:**
- HP ≤ 30 の場合、Tick処理できぶんの自然低下速度が2倍になる

---

## 6. ローカルフォールバック（定型文）

オフライン時またはAPI呼び出し失敗時に表示する定型文。
パラメータ状態に応じて選択する。

| 条件 | 定型文例 |
|------|---------|
| きぶん ≥ 70 | 「ねぇねぇ、今日もいい天気だね〜」 |
| きぶん 30〜69 | 「…ふぁ〜、ちょっと眠いかも」 |
| きぶん ≤ 29 | 「……」 |
| HP ≤ 30 | 「うぅ…ちょっとしんどい…」 |
| きずな ≥ 80 | 「えへへ、一緒にいると落ち着くなぁ」 |

定型文は進化段階・進化タイプごとにバリエーションを持たせる（拡張可能な設計）。
- たまご: 1セット
- こども: 3セット（brain, heart, mood）
- おとな: 7セット（sage, scholar, soulmate, cheerful, trickster, free-spirit, stray）
- stray（育成失敗）は独自のそっけない定型文セットを持つ
