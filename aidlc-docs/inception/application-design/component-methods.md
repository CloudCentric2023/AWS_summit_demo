# devpet コンポーネントメソッド定義

> **Note**: ビジネスロジックの詳細（パラメータ計算式、進化条件等）は機能設計（CONSTRUCTION phase）で定義します。
> ここではメソッドシグネチャと高レベルの目的のみを記載します。

---

## 1. TUIクライアント

### 1.1 PetEngine

```go
// Init はローカルデータを読み込み、経過時間計算を適用してペット状態を初期化する
func (e *PetEngine) Init() (*shared.Pet, error)

// Talk は「話しかける」アクションを処理し、EXP獲得とパラメータ変動を適用する
func (e *PetEngine) Talk() (*TalkResult, error)

// Tick は時間経過によるパラメータ自然変動を適用する（TUI描画ループから定期呼び出し）
func (e *PetEngine) Tick() error

// CheckEvolution はレベルアップ・進化条件を判定する
func (e *PetEngine) CheckEvolution() (*EvolutionResult, error)

// CheckDeath は死亡条件（HP=0）を判定する
func (e *PetEngine) CheckDeath() bool

// GetStatus は現在のペット状態を返す
func (e *PetEngine) GetStatus() *shared.Pet
```

### 1.2 TimeCalculator

```go
// CalcElapsed は前回操作時刻からの経過時間に基づくパラメータ変動を計算する
func (c *TimeCalculator) CalcElapsed(pet *shared.Pet, now time.Time) *ParamDelta

// CalcDeath は経過時間に基づく死亡判定を行う
func (c *TimeCalculator) CalcDeath(pet *shared.Pet, now time.Time) bool
```

### 1.3 ExpCalculator

```go
// CalcTalkExp は「話しかける」によるEXP獲得量を計算する
func (c *ExpCalculator) CalcTalkExp(pet *shared.Pet) int

// CalcHookExp はAIツール連携（hook）によるEXP獲得量を計算する
func (c *ExpCalculator) CalcHookExp(pet *shared.Pet) int

// CalcLevelUp はEXPに基づくレベルアップ判定を行う
func (c *ExpCalculator) CalcLevelUp(pet *shared.Pet) (newLevel int, leveledUp bool)
```

### 1.4 EvolutionManager

```go
// CheckEvolution はパラメータ傾向に基づく進化先を判定する
func (m *EvolutionManager) CheckEvolution(pet *shared.Pet) (*EvolutionResult, error)

// GetEvolutionCandidates は現在のパラメータで到達可能な進化先一覧を返す
func (m *EvolutionManager) GetEvolutionCandidates(pet *shared.Pet) []EvolutionCandidate

// GetAsciiArt は進化段階に応じたAAを返す
func (m *EvolutionManager) GetAsciiArt(stage string) string
```

### 1.5 LocalStorage

```go
// Save はペットデータをローカルファイルに保存する
func (s *LocalStorage) Save(pet *shared.Pet) error

// Load はローカルファイルからペットデータを読み込む
func (s *LocalStorage) Load() (*shared.Pet, error)

// SaveConfig は設定データを保存する
func (s *LocalStorage) SaveConfig(config *Config) error

// LoadConfig は設定データを読み込む
func (s *LocalStorage) LoadConfig() (*Config, error)
```

### 1.6 APIClient

```go
// Talk はバックエンドのAI応答APIを呼び出す
func (c *APIClient) Talk(pet *shared.Pet) (*shared.TalkResponse, error)

// SaveToCloud はペットデータをクラウドに保存する
func (c *APIClient) SaveToCloud(pet *shared.Pet) error

// LoadFromCloud はクラウドからペットデータを読み込む
func (c *APIClient) LoadFromCloud() (*shared.Pet, error)

// IsOnline はバックエンドへの接続可否を確認する
func (c *APIClient) IsOnline() bool
```

### 1.7 AuthClient

```go
// StartDeviceAuth はデバイスコード認証フローを開始する（ブラウザを開く）
func (c *AuthClient) StartDeviceAuth() (*DeviceAuthResult, error)

// PollDeviceAuth はデバイス認証の完了をポーリングする
func (c *AuthClient) PollDeviceAuth(deviceCode string) (*AuthTokens, error)

// RefreshToken はアクセストークンをリフレッシュする
func (c *AuthClient) RefreshToken() (*AuthTokens, error)

// IsAuthenticated は認証済みかどうかを返す
func (c *AuthClient) IsAuthenticated() bool

// Logout はログアウト処理を行う
func (c *AuthClient) Logout() error
```

---

## 2. AWSバックエンド

### 2.1 TalkHandler

```go
// Handle は「話しかける」APIリクエストを処理する
func (h *TalkHandler) Handle(ctx context.Context, req *shared.TalkRequest) (*shared.TalkResponse, error)
```

### 2.2 SaveHandler

```go
// Handle はセーブAPIリクエストを処理する
func (h *SaveHandler) Handle(ctx context.Context, req *shared.SaveRequest) (*shared.SaveResponse, error)
```

### 2.3 LoadHandler

```go
// Handle はロードAPIリクエストを処理する
func (h *LoadHandler) Handle(ctx context.Context, req *shared.LoadRequest) (*shared.LoadResponse, error)
```

### 2.4 AIService

```go
// GenerateResponse はペット状態に基づいてAI応答を生成する
func (s *AIService) GenerateResponse(ctx context.Context, pet *shared.Pet) (string, error)

// GetPromptTemplate はBedrock Prompt Managementからプロンプトテンプレートを取得する
func (s *AIService) GetPromptTemplate(ctx context.Context, stage string) (string, error)

// BuildPrompt はパラメータを埋め込んだ最終プロンプトを構築する
func (s *AIService) BuildPrompt(template string, pet *shared.Pet) string
```

### 2.5 PetService

```go
// Save はペットデータをDynamoDBに保存する
func (s *PetService) Save(ctx context.Context, userID string, pet *shared.Pet) error

// Load はDynamoDBからアクティブなペットデータを取得する
func (s *PetService) Load(ctx context.Context, userID string) (*shared.Pet, error)

// MarkDead はペットを死亡状態に更新する
func (s *PetService) MarkDead(ctx context.Context, userID string, petID string) error
```

### 2.6 Repositories

```go
// PetRepository
func (r *PetRepository) Put(ctx context.Context, userID string, pet *shared.Pet) error
func (r *PetRepository) Get(ctx context.Context, userID string, petID string) (*shared.Pet, error)
func (r *PetRepository) GetActive(ctx context.Context, userID string) (*shared.Pet, error)
func (r *PetRepository) ListDead(ctx context.Context, userID string) ([]*shared.Pet, error)
func (r *PetRepository) Delete(ctx context.Context, userID string, petID string) error
```

---

## 3. 共有パッケージ (devpet-shared)

### 3.1 Models

```go
type Pet struct {
    PetID             string    `json:"petId"`
    Name              string    `json:"name"`
    Stage             string    `json:"stage"`     // "egg", "child", "adult"
    Level             int       `json:"level"`
    Exp               int       `json:"exp"`
    HP                int       `json:"hp"`
    Kashikosa         int       `json:"kashikosa"` // かしこさ (0-100)
    Kizuna            int       `json:"kizuna"`    // きずな (0-100)
    Kibun             int       `json:"kibun"`     // きぶん (0-100)
    LastInteractionAt time.Time `json:"lastInteractionAt"`
    IsDead            bool      `json:"isDead"`
    DiedAt            *time.Time `json:"diedAt,omitempty"`
    CauseOfDeath      string    `json:"causeOfDeath,omitempty"`
    CreatedAt         time.Time `json:"createdAt"`
    UpdatedAt         time.Time `json:"updatedAt"`
}

type TalkRequest struct {
    PetID string     `json:"petId"`
    Pet   *Pet       `json:"pet"`
}

type TalkResponse struct {
    Message      string         `json:"message"`
    ExpGained    int            `json:"expGained"`
    ParamChanges map[string]int `json:"paramChanges"`
}

type SaveRequest struct {
    Pet *Pet `json:"pet"`
}

type SaveResponse struct {
    Success bool   `json:"success"`
    Message string `json:"message"`
}

type LoadRequest struct{}

type LoadResponse struct {
    Pet *Pet `json:"pet"`
}
```

### 3.2 Validation

```go
// ValidatePet はペットデータの整合性を検証する
func ValidatePet(pet *Pet) error

// ClampParam はパラメータ値を0-100の範囲にクランプする
func ClampParam(value int) int
```
