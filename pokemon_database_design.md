# ポケモン対戦支援アプリ データベース設計書

## 1. システム概要
本システムは、ユーザが手持ちのポケモンを登録し、対戦相手のポケモン情報を入力することで、
タイプ相性や能力値（特にすばやさ）を考慮して最適な手持ちポケモンを提示するアプリケーションである。

---

## 2. システム構成図

```text
[ユーザ]
   |
   | HTTP
   v
[Webアプリケーション]
 (登録・検索・判定ロジック)
   |
   | SQL
   v
[データベース]
 PostgreSQL / MySQL
```

---

## 3. ER図設計（主キー・外部キー・制約）

### 3.1 users（ユーザ）
| 属性名 | 型 | 制約 |
|---|---|---|
| user_id | INT | PK |
| user_name | VARCHAR | NOT NULL |

### 3.2 types（タイプ）
| 属性名 | 型 | 制約 |
|---|---|---|
| type_id | INT | PK |
| type_name | VARCHAR | UNIQUE, NOT NULL |

### 3.3 pokemon_master（ポケモンマスタ）
| 属性名 | 型 | 制約 |
|---|---|---|
| pokemon_id | INT | PK |
| pokemon_name | VARCHAR | NOT NULL |
| type_id | INT | FK → types(type_id) |
| attack | INT | NOT NULL |
| defense | INT | NOT NULL |
| speed | INT | NOT NULL |

### 3.4 user_pokemon（手持ちポケモン）
| 属性名 | 型 | 制約 |
|---|---|---|
| user_pokemon_id | INT | PK |
| user_id | INT | FK → users(user_id) |
| pokemon_id | INT | FK → pokemon_master(pokemon_id) |

### 3.5 enemy_pokemon（対戦相手ポケモン）
| 属性名 | 型 | 制約 |
|---|---|---|
| enemy_id | INT | PK |
| pokemon_id | INT | FK → pokemon_master(pokemon_id) |

### 3.6 type_effectiveness（タイプ相性）
| 属性名 | 型 | 制約 |
|---|---|---|
| attack_type_id | INT | PK, FK → types(type_id) |
| defense_type_id | INT | PK, FK → types(type_id) |
| effectiveness | DECIMAL | NOT NULL |

---

## 4. トランザクション処理設計

### 手持ちポケモン登録処理
```sql
BEGIN;

INSERT INTO user_pokemon(user_id, pokemon_id)
VALUES (1, 25);

COMMIT;
```

### エラー発生時
```sql
ROLLBACK;
```

トランザクションにより、途中失敗時でもデータの整合性が保たれる。

---

## 5. JOIN・副問い合わせ設計

### 5.1 JOINを用いた有利ポケモン検索
```sql
SELECT pm.pokemon_name, pm.attack, pm.speed, te.effectiveness
FROM user_pokemon up
JOIN pokemon_master pm ON up.pokemon_id = pm.pokemon_id
JOIN enemy_pokemon ep ON ep.enemy_id = 1
JOIN pokemon_master epm ON ep.pokemon_id = epm.pokemon_id
JOIN type_effectiveness te
  ON pm.type_id = te.attack_type_id
 AND epm.type_id = te.defense_type_id
WHERE up.user_id = 1
ORDER BY te.effectiveness DESC, pm.speed DESC;
```

### 5.2 副問い合わせの例（平均以上のすばやさ）
```sql
SELECT pokemon_name
FROM pokemon_master
WHERE speed >= (
  SELECT AVG(speed)
  FROM pokemon_master
);
```

---

## 6. 正規化設計

### 第1正規形
- 繰り返し属性を排除
- 1セル1値を保証

### 第2正規形
- 複合主キーに完全関数従属しない属性を分離
- タイプ相性を独立テーブルとして定義

### 第3正規形
- 推移的関数従属を排除
- タイプ情報とポケモン情報を分離

本設計は第3正規形を満たし、冗長性や更新異常を防いでいる。

---

## 7. まとめ
本データベース設計により、ユーザの手持ちポケモンと対戦相手ポケモンを効率的に管理し、
JOINや副問い合わせを用いてタイプ相性と能力値に基づいた最適なポケモン選択を実現できる。
