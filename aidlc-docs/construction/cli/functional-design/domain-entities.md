# cli — ドメインエンティティ定義

## cli固有の型定義

### TalkResult
```go
type TalkResult struct {
    Message      string         // AI応答テキスト or ローカル定型文
    ExpGained    int            // 獲得EXP
    ParamChanges map[string]int // パラメータ変動量
    LeveledUp    bool           // レベルアップしたか
    Evolved      bool           // 進化したか
    NewStage     shared.Stage   // 進化後のStage（進化した場合）
    EvoType      string         // 進化タイプ（進化した場合）
    IsLocal      bool           // ローカルフォールバックか
}
```

### ParamDelta（cli内部用、詳細版）
```go
type ParamDelta struct {
    HP        float64 // 小数点精度で計算し、適用時にintに変換
    Kashikosa float64
    Kizuna    float64
    Kibun     float64
}
```

### EvolutionResult
```go
type EvolutionResult struct {
    Evolved   bool
    NewStage  shared.Stage
    EvoType   string
    OldStage  shared.Stage
}
```

### EvolutionCandidate
```go
type EvolutionCandidate struct {
    EvoType     string
    ParamName   string // 判定パラメータ名
    CurrentVal  int    // 現在値
    RequiredVal int    // 必要値（参考: 他パラメータより高い必要がある）
}
```

### Config（設定データ）
```go
type Config struct {
    APIEndpoint  string `json:"apiEndpoint"`
    AccessToken  string `json:"accessToken,omitempty"`
    RefreshToken string `json:"refreshToken,omitempty"`
    TokenExpiry  *time.Time `json:"tokenExpiry,omitempty"`
    ThemeName    string `json:"themeName"`
}
```

### DeviceAuthResult
```go
type DeviceAuthResult struct {
    DeviceCode      string
    UserCode        string
    VerificationURI string
    ExpiresIn       int
    Interval        int
}
```

### AuthTokens
```go
type AuthTokens struct {
    AccessToken  string
    RefreshToken string
    ExpiresIn    int
}
```

---

## テスト可能プロパティ（PBT-01）

| コンポーネント | プロパティカテゴリ | プロパティ説明 |
|--------------|-----------------|-------------|
| **PetEngine** | | |
| PetEngine.Talk | Invariant | Talk後のEXPは常にTalk前のEXP以上 |
| PetEngine.Talk | Invariant | Talk後の全パラメータは0〜100の範囲内 |
| PetEngine.Tick | Invariant | Tick後の全パラメータは0〜100の範囲内 |
| PetEngine.Init | Invariant | Init後のペットは常に有効な状態（ValidatePet通過） |
| **TimeCalculator** | | |
| TimeCalculator.CalcDeath | Invariant | 経過時間 ≥ 24h → 常にtrue |
| TimeCalculator.CalcDeath | Invariant | 経過時間 < 24h → 常にfalse |
| TimeCalculator.CalcElapsed | Invariant | 経過時間0 → ParamDelta全て0 |
| TimeCalculator.CalcElapsed | Invariant | 経過時間が長いほどパラメータ低下量が大きい（単調性） |
| **ExpCalculator** | | |
| ExpCalculator.CalcTalkExp | Invariant | 常に100を返す |
| ExpCalculator.CalcHookExp | Invariant | 常に10を返す |
| ExpCalculator.CalcLevelUp | Invariant | レベルは下がらない（newLevel ≥ pet.Level） |
| **EvolutionManager** | | |
| EvolutionManager.CheckEvolution | Invariant | 進化はStageが進む方向のみ（egg→child→adult） |
| EvolutionManager.CheckEvolution | Invariant | 同じパラメータ値・同じこどもタイプなら常に同じ進化タイプ（決定的） |
| EvolutionManager.CheckEvolution | Commutativity | パラメータの設定順序に関わらず同じ進化結果 |
| EvolutionManager.CheckEvolution | Invariant | おとな進化は7種類（sage, scholar, soulmate, cheerful, trickster, free-spirit, stray）のいずれか |
| EvolutionManager.CheckEvolution | Invariant | かしこさ≤30 かつ きずな≤30 かつ きぶん≤30 → 常にstray |
| EvolutionManager.CheckEvolution | Invariant | いずれかのパラメータ（かしこさ/きずな/きぶん）が31以上 → strayにならない |
| EvolutionManager.CheckEvolution | Invariant | こども進化はHP値に依存しない（HPを変えても結果が変わらない） |
| EvolutionManager.CheckEvolution | Invariant | おとな進化はHP値に依存しない（HPを変えても結果が変わらない） |
| **LocalStorage** | | |
| LocalStorage Save/Load | Round-trip | Save(pet) → Load() == pet |
| Config Save/Load | Round-trip | SaveConfig(cfg) → LoadConfig() == cfg |
