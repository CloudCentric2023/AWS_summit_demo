# backend — ビジネスロジックモデル

## 1. TalkHandler（話しかけAPI）

### 処理フロー
```
入力: TalkRequest (petId, pet)
出力: TalkResponse (message, expGained, paramChanges)

1. リクエストバリデーション（shared.ValidatePet）
2. 認証トークン検証（Cognito JWT）
3. ユーザーIDとpetIdの所有権確認（SECURITY-08: IDOR防止）
4. AIService.GenerateResponse() でAI応答を生成
5. EXP獲得量を計算（固定: 100）
6. パラメータ変動量を計算（話しかけ変動ルール準拠）
7. TalkResponse を返却

エラー時:
- バリデーションエラー → 400 Bad Request
- 認証エラー → 401 Unauthorized
- 所有権エラー → 403 Forbidden
- Bedrock呼び出しエラー → 500 Internal Server Error（詳細はログのみ）
```

---

## 2. SaveHandler（セーブAPI）

### 処理フロー
```
入力: SaveRequest (pet)
出力: SaveResponse (success, message)

1. リクエストバリデーション（shared.ValidatePet）
2. 認証トークン検証（Cognito JWT）
3. ユーザーIDの抽出（JWTのsub claim）
4. PetService.Save() でDynamoDBに保存
5. SaveResponse を返却

エラー時:
- バリデーションエラー → 400 Bad Request
- 認証エラー → 401 Unauthorized
- DynamoDB書き込みエラー → 500 Internal Server Error
```

---

## 3. LoadHandler（ロードAPI）

### 処理フロー
```
入力: LoadRequest (petId from path parameter)
出力: LoadResponse (pet)

1. 認証トークン検証（Cognito JWT）
2. ユーザーIDの抽出（JWTのsub claim）
3. PetService.Load() でDynamoDBからアクティブなペットを取得
4. 所有権確認（取得したペットのuserIdとJWTのsubが一致）
5. LoadResponse を返却

エラー時:
- 認証エラー → 401 Unauthorized
- ペット未存在 → 404 Not Found
- DynamoDB読み込みエラー → 500 Internal Server Error
```

---

## 4. AuthHandler（認証API）

### デバイスコード認証開始
```
エンドポイント: POST /auth/device
入力: なし
出力: DeviceAuthResponse

1. Cognitoにデバイスコード発行をリクエスト
2. device_code, user_code, verification_uri を返却
3. TUIはverification_uriをブラウザで開く

※ このエンドポイントは認証不要
```

### デバイス認証ポーリング
```
エンドポイント: POST /auth/device/poll
入力: DeviceAuthPollRequest (deviceCode)
出力: DeviceAuthPollResponse

1. device_codeでCognitoに認証状態を問い合わせ
2. 状態に応じてレスポンス:
   - pending: ユーザーがまだ承認していない
   - complete: 認証完了、access_token + refresh_token を返却
   - expired: デバイスコードの有効期限切れ

※ このエンドポイントは認証不要
```

---

## 5. AIService（AI応答生成サービス）

### GenerateResponse フロー
```
入力: Pet
出力: string (AI応答テキスト、30文字程度)

1. PromptClient.GetPrompt() でBedrock Prompt Managementからテンプレート取得
   - テンプレートは進化段階（stage）と進化タイプ（evolutionType）に応じて選択
   - たまご: `devpet-egg-prompt`
   - こども/おとな: `devpet-{stage}-{evolutionType}-prompt`
2. BuildPrompt() でパラメータを埋め込み
3. BedrockClient.InvokeModel() でClaude Haikuに応答生成をリクエスト
4. 応答テキストを返却（30文字程度に制限）
```

### BuildPrompt — トーン制御ロジック

**提案: パラメータ値を直接プロンプトに埋め込み、AIに自然なトーン調整を委ねる方式**

プロンプトテンプレート構造:
```
あなたは「{petName}」という名前のバーチャルペットです。
進化段階: {stage}（{evolutionType}タイプ）
レベル: {level}

現在の状態:
- たいりょく(HP): {hp}/100
- かしこさ: {kashikosa}/100
- きずな: {kizuna}/100
- きぶん: {kibun}/100

以下のルールに従って、飼い主に話しかけてください:
- 30文字以内の日本語で1文だけ返答してください
- パラメータの値に応じて自然にトーンを変えてください:
  - きずなが高い(70+)ほど親密で甘えた口調
  - きずなが低い(30-)ほどよそよそしい口調
  - きぶんが高い(70+)ほどハイテンション
  - きぶんが低い(30-)ほど無口・ぼそっと
  - かしこさが高い(70+)ほど博識で的確
  - HPが低い(30-)ほどぐったり・元気がない
- 特殊な組み合わせ:
  - きずな高+かしこさ高 → 頼れる相棒感
  - きぶん低+きずな高 → 「だるいけど…あなただから」感
  - かしこさ高+きぶん低 → ツンデレ感
```

**この方式の根拠:**
- パラメータ値を数値で直接渡すことで、AIが自然にグラデーションのあるトーン調整を行える
- 2段階や3段階の閾値で切り替えるよりも、より自然で多様な応答が生成される
- Bedrock Prompt Managementの変数機能でパラメータ値を動的に埋め込める
- プロンプトテンプレートの更新だけでトーン制御を調整可能（コード変更不要）

---

## 6. PetService（ペットデータサービス）

### Save
```
入力: userID, Pet
出力: error

1. shared.ValidatePet() でバリデーション
2. Pet.UpdatedAt を現在時刻に更新
3. PetRepository.Put() でDynamoDBに書き込み
```

### Load
```
入力: userID
出力: *Pet, error

1. PetRepository.GetActive() でアクティブなペット（isDead=false）を取得
2. ペットが存在しない場合 → nil, nil（404として処理）
3. ペットが存在する場合 → Pet を返却
```

### MarkDead
```
入力: userID, petID
出力: error

1. PetRepository.Get() でペットを取得
2. Pet.IsDead = true
3. Pet.DiedAt = 現在時刻
4. PetRepository.Put() で更新
```

---

## テスト可能プロパティ（PBT-01）

| コンポーネント | プロパティカテゴリ | プロパティ説明 |
|--------------|-----------------|-------------|
| **TalkHandler** | | |
| TalkHandler | Invariant | 有効なリクエストに対して常にTalkResponseを返す |
| TalkHandler | Invariant | レスポンスのExpGainedは常に正の値 |
| **AIService** | | |
| AIService.BuildPrompt | Invariant | 任意の有効なPetに対して空でないプロンプト文字列を生成 |
| AIService.BuildPrompt | Invariant | プロンプトに全パラメータ値が含まれる |
| **PetService** | | |
| PetService Save/Load | Round-trip | Save(user, pet) → Load(user) で同じペットが取得できる |
| PetService.Save | Invariant | Save後のUpdatedAtは常にSave前以降の時刻 |
| **PetRepository** | | |
| PetRepository Put/Get | Round-trip | Put(user, pet) → Get(user, petId) == pet |
| PetRepository.GetActive | Invariant | 返却されるペットのisDeadは常にfalse |
| PetRepository.ListDead | Invariant | 返却される全ペットのisDeadは常にtrue |
| **Input Validation** | | |
| Request Validation | Invariant | 無効なリクエストは常にエラーを返す（400） |
| Request Validation | Invariant | 有効なリクエストはエラーを返さない |
