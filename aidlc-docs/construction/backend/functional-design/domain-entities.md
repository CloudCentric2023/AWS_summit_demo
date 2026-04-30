# backend — ドメインエンティティ定義

## backend固有の型定義

### DynamoDB Item構造

#### PetItem（DynamoDB保存形式）
```go
type PetItem struct {
    PK                string `dynamodbav:"PK"`                // "USER#{userId}"
    SK                string `dynamodbav:"SK"`                // "PET#{petId}"
    Name              string `dynamodbav:"name"`
    Stage             string `dynamodbav:"stage"`
    EvolutionType     string `dynamodbav:"evolutionType"`
    Level             int    `dynamodbav:"level"`
    Exp               int    `dynamodbav:"exp"`
    HP                int    `dynamodbav:"hp"`
    Kashikosa         int    `dynamodbav:"kashikosa"`
    Kizuna            int    `dynamodbav:"kizuna"`
    Kibun             int    `dynamodbav:"kibun"`
    LastInteractionAt string `dynamodbav:"lastInteractionAt"` // ISO 8601
    IsDead            bool   `dynamodbav:"isDead"`
    DiedAt            string `dynamodbav:"diedAt,omitempty"`   // ISO 8601
    CauseOfDeath      string `dynamodbav:"causeOfDeath,omitempty"`
    CreatedAt         string `dynamodbav:"createdAt"`          // ISO 8601
    UpdatedAt         string `dynamodbav:"updatedAt"`          // ISO 8601
    
    // GSI1用
    GSI1PK            string `dynamodbav:"GSI1PK"`            // "USER#{userId}"
    GSI1SK            string `dynamodbav:"GSI1SK"`            // "ALIVE#" or "DEAD#{diedAt}"
}
```

#### UserItem（DynamoDB保存形式）
```go
type UserItem struct {
    PK          string `dynamodbav:"PK"`          // "USER#{userId}"
    SK          string `dynamodbav:"SK"`          // "PROFILE"
    Email       string `dynamodbav:"email"`
    CreatedAt   string `dynamodbav:"createdAt"`   // ISO 8601
    LastLoginAt string `dynamodbav:"lastLoginAt"` // ISO 8601
}
```

### 変換関数

```go
// PetItem ↔ shared.Pet の変換
func PetItemFromPet(userID string, pet *shared.Pet) *PetItem
func (item *PetItem) ToPet() *shared.Pet

// UserItem ↔ User の変換
func UserItemFromUser(user *User) *UserItem
func (item *UserItem) ToUser() *User
```

### Lambda イベント型

```go
// API Gateway Proxy統合のリクエスト/レスポンス
// aws-lambda-go の events パッケージを使用
// events.APIGatewayProxyRequest
// events.APIGatewayProxyResponse
```

### 構造化ログ型

```go
type LogEntry struct {
    Timestamp string `json:"timestamp"`
    RequestID string `json:"requestId"`
    Level     string `json:"level"`     // "INFO", "WARN", "ERROR"
    Message   string `json:"message"`
    ErrorCode string `json:"errorCode,omitempty"`
    UserID    string `json:"userId,omitempty"`
    PetID     string `json:"petId,omitempty"`
    Duration  int64  `json:"durationMs,omitempty"`
}
```

---

## DynamoDB アクセスパターン

| アクセスパターン | テーブル/GSI | PK | SK | 操作 |
|---------------|------------|----|----|------|
| ペット保存 | Pets | USER#{userId} | PET#{petId} | PutItem |
| ペット取得 | Pets | USER#{userId} | PET#{petId} | GetItem |
| アクティブペット取得 | GSI1 | USER#{userId} | begins_with("ALIVE#") | Query |
| 死亡ペット一覧 | GSI1 | USER#{userId} | begins_with("DEAD#") | Query |
| ユーザー保存 | Users | USER#{userId} | PROFILE | PutItem |
| ユーザー取得 | Users | USER#{userId} | PROFILE | GetItem |
