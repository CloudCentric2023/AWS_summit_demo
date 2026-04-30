# shared — ビジネスロジックモデル

## 概要
sharedユニットはビジネスロジックの実装を持たない（型定義・バリデーション・定数のみ）。
ビジネスロジックの実装は cli（ローカル計算）と backend（API処理）に分散する。

sharedが提供するのは以下のみ:
1. **ドメインエンティティ**: → `domain-entities.md` 参照
2. **バリデーション関数**: パラメータ範囲チェック、データ整合性検証
3. **定数**: 進化段階名、パラメータ上限値、EXP計算係数等

## バリデーション関数

### ValidatePet
```
入力: Pet
出力: error (nil = 有効)

ルール:
1. PetID が空でないこと
2. Name が 1〜20文字であること
3. Stage が有効な値であること（egg/child/adult）
4. Level が 1以上であること
5. Exp が 0以上であること
6. HP, Kashikosa, Kizuna, Kibun が 0〜100の範囲であること
7. IsDead=true の場合、DiedAt が nil でないこと
8. LastInteractionAt が未来の時刻でないこと
```

### ClampParam
```
入力: value (int)
出力: int (0〜100にクランプされた値)

ルール:
- value < 0 → 0
- value > 100 → 100
- それ以外 → value
```

### CalcLevel
```
入力: exp (int, 累積EXP)
出力: int (レベル)

ルール:
- 差分テーブルから累積EXPを算出し、exp以下の最大レベルを返す
- exp < 0 → 1
```

### CalcRequiredExp
```
入力: level (int)
出力: int (そのレベルに到達するために必要な累積EXP)

ルール:
- 差分テーブルの合計で算出
- 差分(N→N+1):
  - N < 10:  100 + 30 × (N - 1)
  - N < 20:  370 + 50 × (N - 10)
  - N ≥ 20:  870 + 80 × (N - 20)
- 累積EXP(Lv) = Σ 差分(1→2) + 差分(2→3) + ... + 差分(Lv-1→Lv)
```

### CalcDeltaExp
```
入力: level (int, 現在のレベル)
出力: int (次のレベルまでの差分EXP)

ルール:
- level < 10:  100 + 30 × (level - 1)
- level < 20:  370 + 50 × (level - 10)
- level ≥ 20:  870 + 80 × (level - 20)
```

## テスト可能プロパティ（PBT-01）

| コンポーネント | プロパティカテゴリ | プロパティ説明 |
|--------------|-----------------|-------------|
| ValidatePet | Invariant | 有効なPetは常にバリデーションを通過する |
| ClampParam | Idempotence | ClampParam(ClampParam(x)) == ClampParam(x) |
| ClampParam | Invariant | 出力は常に 0 ≤ result ≤ 100 |
| CalcLevel | Invariant | CalcLevel(exp) ≥ 1 for all exp ≥ 0 |
| CalcLevel | Invariant | CalcLevel(CalcRequiredExp(lv)) == lv |
| CalcRequiredExp | Invariant | CalcRequiredExp(lv) は lv に対して単調増加 |
| CalcDeltaExp | Invariant | CalcDeltaExp(lv) は lv に対して単調増加（差分は常に前レベルより大きい） |
| CalcDeltaExp | Invariant | CalcDeltaExp(lv) > 0 for all lv ≥ 1 |
| Pet JSON | Round-trip | serialize(deserialize(json)) == json（有効なPetデータ） |
| Pet JSON | Round-trip | deserialize(serialize(pet)) == pet |
