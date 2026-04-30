# shared — ドメインエンティティ定義

## 1. Pet（ペット）

ペットの全状態を表すコアエンティティ。TUIクライアントとバックエンドの両方で使用される。

```go
type Pet struct {
    PetID             string     `json:"petId"`
    Name              string     `json:"name"`
    Stage             Stage      `json:"stage"`
    EvolutionType     string     `json:"evolutionType"`     // 進化タイプ名（例: "power", "brain", "heart"）
    Level             int        `json:"level"`
    Exp               int        `json:"exp"`               // 累積EXP
    HP                int        `json:"hp"`                 // 0-100
    Kashikosa         int        `json:"kashikosa"`          // かしこさ 0-100
    Kizuna            int        `json:"kizuna"`             // きずな 0-100
    Kibun             int        `json:"kibun"`              // きぶん 0-100
    LastInteractionAt time.Time  `json:"lastInteractionAt"`
    IsDead            bool       `json:"isDead"`
    DiedAt            *time.Time `json:"diedAt,omitempty"`
    CauseOfDeath      string     `json:"causeOfDeath,omitempty"`
    CreatedAt         time.Time  `json:"createdAt"`
    UpdatedAt         time.Time  `json:"updatedAt"`
}
```

### 初期値（たまご生成時）

| パラメータ | 初期値 | 根拠 |
|-----------|--------|------|
| Stage | `Egg` | たまごからスタート |
| EvolutionType | `""` (空) | 進化前は未決定 |
| Level | 1 | レベル1からスタート |
| Exp | 0 | EXP 0からスタート |
| HP | 100 | 満タンからスタート |
| Kashikosa | 0 | ゼロからスタート |
| Kizuna | 0 | ゼロからスタート |
| Kibun | 50 | 中間値（生まれたてはニュートラル） |

---

## 2. Stage（進化段階）

```go
type Stage string

const (
    StageEgg   Stage = "egg"
    StageChild Stage = "child"
    StageAdult Stage = "adult"
)
```

---

## 3. EvolutionType（進化タイプ）

各段階で3種類に分岐する。進化判定には **かしこさ・きずな・きぶん** の3パラメータを使用する。
HP（たいりょく）は進化判定に使用しない（HPは生存指標であり、育成の方向性を表さないため）。

### こども段階（3種類）

| タイプ名 | 判定条件 | キャラクター特徴 |
|---------|---------|----------------|
| `brain` | かしこさが最も高い | 物知り、好奇心旺盛 |
| `heart` | きずなが最も高い | 甘えん坊、人懐っこい |
| `mood` | きぶんが最も高い | 気分屋、自由奔放 |

### おとな段階（7種類）

こども段階の進化タイプに応じて、それぞれ2方向に分岐する（計6種）。加えて、育成失敗パターンが1種。

#### brain（物知りタイプ）からの分岐

| タイプ名 | 判定条件 | キャラクター特徴 |
|---------|---------|----------------|
| `sage` | きずなが きぶん以上 | 賢者。知恵と絆で的確なアドバイスをくれる、頼れる知恵袋 |
| `scholar` | きぶんが きずなより高い | 学者。知識と気分で気まぐれな天才肌 |

#### heart（甘えん坊タイプ）からの分岐

| タイプ名 | 判定条件 | キャラクター特徴 |
|---------|---------|----------------|
| `soulmate` | かしこさが きぶん以上 | 魂の伴侶。深い絆と知恵で心を読む、最高の理解者 |
| `cheerful` | きぶんが かしこさより高い | 元気な甘えん坊。絆と気分で明るく寄り添う、ムードメーカー |

#### mood（気分屋タイプ）からの分岐

| タイプ名 | 判定条件 | キャラクター特徴 |
|---------|---------|----------------|
| `trickster` | かしこさが きずな以上 | トリックスター。気分と知恵で予測不能な策士 |
| `free-spirit` | きずなが かしこさより高い | 自由人。気分と絆で気ままに寄り添う相棒 |

#### 育成失敗パターン

| タイプ名 | 判定条件 | キャラクター特徴 |
|---------|---------|----------------|
| `stray` | かしこさ ≤ 30 かつ きずな ≤ 30 かつ きぶん ≤ 30 | はぐれ者。放置されすぎて野生化した、気まぐれな存在 |

#### おとな進化の判定フロー

```
1. 育成失敗チェック: かしこさ ≤ 30 かつ きずな ≤ 30 かつ きぶん ≤ 30 → stray
2. こども段階の進化タイプに応じて分岐:
   - brain → きずな ≥ きぶん なら sage、そうでなければ scholar
   - heart → かしこさ ≥ きぶん なら soulmate、そうでなければ cheerful
   - mood  → かしこさ ≥ きずな なら trickster、そうでなければ free-spirit
```

### 同率時のタイブレーク

こども段階: パラメータが同率の場合の優先順位: きずな > かしこさ > きぶん
おとな段階: 各分岐の判定条件で「以上（≥）」を使用し、同率時は前者を優先

**根拠**: devpetのコンセプト「話し相手」に最も合致するきずなを優先。

---

## 4. TalkRequest / TalkResponse

```go
type TalkRequest struct {
    PetID string `json:"petId"`
    Pet   *Pet   `json:"pet"`
}

type TalkResponse struct {
    Message      string         `json:"message"`      // AI応答テキスト（30文字程度）
    ExpGained    int            `json:"expGained"`     // 獲得EXP
    ParamChanges map[string]int `json:"paramChanges"`  // パラメータ変動量
}
```

---

## 5. SaveRequest / SaveResponse

```go
type SaveRequest struct {
    Pet *Pet `json:"pet"`
}

type SaveResponse struct {
    Success bool   `json:"success"`
    Message string `json:"message"`
}
```

---

## 6. LoadRequest / LoadResponse

```go
type LoadRequest struct{}

type LoadResponse struct {
    Pet *Pet `json:"pet"`
}
```

---

## 7. DeviceAuthRequest / DeviceAuthResponse

```go
type DeviceAuthRequest struct{}

type DeviceAuthResponse struct {
    DeviceCode      string `json:"deviceCode"`
    UserCode        string `json:"userCode"`
    VerificationURI string `json:"verificationUri"`
    ExpiresIn       int    `json:"expiresIn"`
    Interval        int    `json:"interval"`
}

type DeviceAuthPollRequest struct {
    DeviceCode string `json:"deviceCode"`
}

type DeviceAuthPollResponse struct {
    Status       string `json:"status"`       // "pending", "complete", "expired"
    AccessToken  string `json:"accessToken,omitempty"`
    RefreshToken string `json:"refreshToken,omitempty"`
    ExpiresIn    int    `json:"expiresIn,omitempty"`
}
```

---

## 8. ParamDelta（パラメータ変動量）

```go
type ParamDelta struct {
    HP        int `json:"hp"`
    Kashikosa int `json:"kashikosa"`
    Kizuna    int `json:"kizuna"`
    Kibun     int `json:"kibun"`
}
```

---

## 9. ErrorResponse

```go
type ErrorResponse struct {
    Code    string `json:"code"`
    Message string `json:"message"`
}
```
