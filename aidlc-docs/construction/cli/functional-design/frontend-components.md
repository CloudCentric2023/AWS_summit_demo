# cli — フロントエンドコンポーネント設計（TUI）

## 1. 画面構成

### 1.1 MainScreen（メイン画面）

bubbletea の Model として実装。全UIコンポーネントを統合する。

**状態:**
```go
type MainScreen struct {
    pet          *shared.Pet
    petEngine    *PetEngine
    apiClient    *APIClient
    authClient   *AuthClient
    
    // UI子コンポーネント
    statusBar    StatusBar
    paramDisplay ParameterDisplay
    petView      PetView
    speechBubble SpeechBubble
    
    // 状態
    isLoading    bool          // API呼び出し中
    lastMessage  string        // 最新の通知メッセージ
    width        int           // ターミナル幅
    height       int           // ターミナル高さ
    theme        *Theme        // カラーテーマ
}
```

**キーバインド:**

| キー | アクション | 条件 |
|------|----------|------|
| `t` | 話しかける | ペット生存中 |
| `s` | セーブ（ローカル + クラウド） | ペット生存中 |
| `l` | ログイン/ログアウト | 常時 |
| `h` | ヘルプ画面へ遷移 | 常時 |
| `?` | 設定画面へ遷移 | 常時 |
| `q` | 終了 | 常時 |
| `y` | 再スタート（新しいたまご） | ペット死亡中 |
| `n` | 終了 | ペット死亡中 |

**レイアウト（80x24）:**
```
+----------------------------------------------------------------------+  行1
| 🐾 devpet - [ペット名]                              [Online/Offline] |  行1
+----------------------------------------------------------------------+  行2
|                                                                      |
|                        [ペットAA表示]                                 |  行3-10
|                                                                      |
|                   +---------------------------+                      |  行11
|                   | [吹き出しテキスト]          |                      |  行12
|                   +---------------------------+                      |  行13
+----------------------------------------------------------------------+  行14
| LV: 5  EXP: 1200/1500  Stage: こども (brain)                        |  行15-16
+----------------------------------------------------------------------+
| HP:       [████████████████████░░░░░░░░░░] 72/100                   |  行17
| かしこさ:  [██████████████████████████░░░░] 85/100                   |  行18
| きずな:    [████████████░░░░░░░░░░░░░░░░░░] 42/100                   |  行19
| きぶん:    [██████████████████░░░░░░░░░░░░] 60/100                   |  行20
+----------------------------------------------------------------------+  行21
| [t] talk  [s] save  [l] login  [h] help  [q] quit                  |  行22
+----------------------------------------------------------------------+
| ✨ レベルアップ！ LV 4 → LV 5                                       |  行23-24
+----------------------------------------------------------------------+
```

---

### 1.2 SettingsScreen（設定画面）

**状態:**
```go
type SettingsScreen struct {
    authClient   *AuthClient
    configMgr    *ConfigManager
    isAuth       bool
    cursor       int      // メニューカーソル位置
    items        []string // メニュー項目
}
```

**メニュー項目:**
- ログイン / ログアウト
- クラウドからロード
- 戻る

**キーバインド:**

| キー | アクション |
|------|----------|
| `↑/↓` or `j/k` | カーソル移動 |
| `Enter` | 選択実行 |
| `Esc` or `q` | メイン画面に戻る |

---

### 1.3 HelpScreen（ヘルプ画面）

**表示内容:**
- キーバインド一覧
- ゲームルール説明（パラメータ、進化、死亡）
- バージョン情報

**キーバインド:**

| キー | アクション |
|------|----------|
| `Esc` or `q` | メイン画面に戻る |
| `↑/↓` | スクロール |

---

### 1.4 DeathScreen（死亡画面）

**表示内容:**
- 墓石AA
- 死亡メッセージ（死因に応じた演出テキスト）
- ペットの最終ステータス（名前、レベル、進化段階）
- 再スタート案内

**死因別メッセージ:**

| 死因 | メッセージ |
|------|----------|
| neglect | 「…お留守の間に、力尽きてしまいました」 |
| exhaustion | 「…がんばりすぎて、疲れちゃったみたい…」 |

---

## 2. UIコンポーネント

### 2.1 StatusBar

```go
type StatusBar struct {
    level     int
    exp       int
    nextExp   int    // 次のレベルまでの必要EXP
    stage     string
    evoType   string
}
```

**表示形式:**
```
LV: 5  EXP: 1200/1500  Stage: こども (brain)
```

### 2.2 ParameterDisplay

```go
type ParameterDisplay struct {
    hp        int
    kashikosa int
    kizuna    int
    kibun     int
    theme     *Theme
}
```

**表示形式（プログレスバー）:**
```
HP:       [████████████████████░░░░░░░░░░] 72/100
```

**カラールール:**
- 値 ≥ 70: Secondary（緑）
- 値 30〜69: デフォルト
- 値 ≤ 29: Danger（赤）

### 2.3 PetView

```go
type PetView struct {
    stage     shared.Stage
    evoType   string
    isDead    bool
    theme     *Theme
}
```

EvolutionManager.GetAsciiArt() を呼び出してAAを取得・表示。

### 2.4 SpeechBubble

```go
type SpeechBubble struct {
    message   string
    maxWidth  int
    isLoading bool  // API呼び出し中は「...」を表示
}
```

**表示形式:**
```
+---------------------------+
| ねぇねぇ、今日もいい天気だね〜 |
+---------------------------+
```

- API呼び出し中: `「...」` をアニメーション表示
- メッセージが長い場合は折り返し

---

## 3. 画面遷移図

```
                    +-------------+
                    | 起動        |
                    +------+------+
                           |
                    +------v------+
                    | データ読込   |
                    +------+------+
                           |
                  +--------+--------+
                  |                 |
           +------v------+  +------v------+
           | MainScreen  |  | DeathScreen |
           +------+------+  +------+------+
                  |                 |
         +-------+-------+        |
         |       |       |   [y]→ 新規ペット生成 → MainScreen
         |       |       |   [n]→ 終了
    [h]  |  [?]  |  [q]  |
         v       v       v
   +-----+  +---+---+  終了
   |Help |  |Settings|
   +-----+  +-------+
   [Esc]→MainScreen
```

---

## 4. bubbletea メッセージ定義

```go
// Tick メッセージ（1秒ごと）
type TickMsg time.Time

// Talk完了メッセージ
type TalkResultMsg struct {
    Message      string
    ExpGained    int
    ParamChanges map[string]int
    Err          error
}

// レベルアップメッセージ
type LevelUpMsg struct {
    OldLevel int
    NewLevel int
}

// 進化メッセージ
type EvolutionMsg struct {
    NewStage shared.Stage
    EvoType  string
}

// 死亡メッセージ
type DeathMsg struct {
    Cause string
}

// セーブ完了メッセージ
type SaveResultMsg struct {
    CloudSaved bool
    Err        error
}

// 認証完了メッセージ
type AuthResultMsg struct {
    Success bool
    Err     error
}

// ターミナルリサイズメッセージ（bubbletea組み込み）
// tea.WindowSizeMsg
```

---

## 5. Theme 構造体

```go
type Theme struct {
    Name       string
    Primary    lipgloss.Color
    Secondary  lipgloss.Color
    Accent     lipgloss.Color
    Danger     lipgloss.Color
    Muted      lipgloss.Color
}

// デフォルトテーマ
var DefaultTheme = Theme{
    Name:      "default",
    Primary:   lipgloss.Color("#7C3AED"),
    Secondary: lipgloss.Color("#10B981"),
    Accent:    lipgloss.Color("#F59E0B"),
    Danger:    lipgloss.Color("#EF4444"),
    Muted:     lipgloss.Color("#6B7280"),
}
```

進化タイプ別テーマは `map[string]Theme` で管理し、拡張可能にする。
