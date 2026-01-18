# ポケモン対戦支援アプリ データベース設計書（MySQL版）

## 1. システム概要
本システムは、ユーザが手持ちのポケモンを登録し、対戦相手のポケモンを入力することで、
タイプ相性および能力値（特にすばやさ）を基に最適な手持ちポケモンを提示する
MySQLデータベースを用いたアプリケーションである。

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
   | SQL (MySQL)
   v
[MySQL データベース]
```

---

## 3. ER図設計（主キー・外部キー・制約）

### 3.1 users（ユーザ）
```sql
CREATE TABLE users (
  user_id INT AUTO_INCREMENT PRIMARY KEY,
  user_name VARCHAR(50) NOT NULL
) ENGINE=InnoDB;
```

### 3.2 types（タイプ）
```sql
CREATE TABLE types (
  type_id INT AUTO_INCREMENT PRIMARY KEY,
  type_name VARCHAR(30) NOT NULL UNIQUE
) ENGINE=InnoDB;
```

### 3.3 pokemon_master（ポケモンマスタ）
```sql
CREATE TABLE pokemon_master (
  pokemon_id INT AUTO_INCREMENT PRIMARY KEY,
  pokemon_name VARCHAR(50) NOT NULL,
  type_id INT NOT NULL,
  attack INT NOT NULL,
  defense INT NOT NULL,
  speed INT NOT NULL,
  CONSTRAINT fk_pokemon_type
    FOREIGN KEY (type_id)
    REFERENCES types(type_id)
) ENGINE=InnoDB;
```

### 3.4 user_pokemon（手持ちポケモン）
```sql
CREATE TABLE user_pokemon (
  user_pokemon_id INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT NOT NULL,
  pokemon_id INT NOT NULL,
  CONSTRAINT fk_up_user
    FOREIGN KEY (user_id)
    REFERENCES users(user_id),
  CONSTRAINT fk_up_pokemon
    FOREIGN KEY (pokemon_id)
    REFERENCES pokemon_master(pokemon_id)
) ENGINE=InnoDB;
```

### 3.5 enemy_pokemon（対戦相手ポケモン）
```sql
CREATE TABLE enemy_pokemon (
  enemy_id INT AUTO_INCREMENT PRIMARY KEY,
  pokemon_id INT NOT NULL,
  CONSTRAINT fk_enemy_pokemon
    FOREIGN KEY (pokemon_id)
    REFERENCES pokemon_master(pokemon_id)
) ENGINE=InnoDB;
```

### 3.6 type_effectiveness（タイプ相性）
```sql
CREATE TABLE type_effectiveness (
  attack_type_id INT NOT NULL,
  defense_type_id INT NOT NULL,
  effectiveness DECIMAL(3,2) NOT NULL,
  PRIMARY KEY (attack_type_id, defense_type_id),
  CONSTRAINT fk_attack_type
    FOREIGN KEY (attack_type_id)
    REFERENCES types(type_id),
  CONSTRAINT fk_defense_type
    FOREIGN KEY (defense_type_id)
    REFERENCES types(type_id)
) ENGINE=InnoDB;
```

---

## 4. トランザクション処理（MySQL）

```sql
START TRANSACTION;

INSERT INTO user_pokemon (user_id, pokemon_id)
VALUES (1, 25);

COMMIT;
```

エラー発生時:
```sql
ROLLBACK;
```

---

## 5. JOIN・副問い合わせ（MySQL）

### 有利なポケモン検索
```sql
SELECT pm.pokemon_name,
       pm.attack,
       pm.speed,
       te.effectiveness
FROM user_pokemon up
JOIN pokemon_master pm
  ON up.pokemon_id = pm.pokemon_id
JOIN enemy_pokemon ep
  ON ep.enemy_id = 1
JOIN pokemon_master epm
  ON ep.pokemon_id = epm.pokemon_id
JOIN type_effectiveness te
  ON pm.type_id = te.attack_type_id
 AND epm.type_id = te.defense_type_id
WHERE up.user_id = 1
ORDER BY te.effectiveness DESC, pm.speed DESC;
```

### 副問い合わせ例
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

- 第1正規形：繰り返し属性なし
- 第2正規形：部分関数従属を排除
- 第3正規形：推移的関数従属を排除

---

## 7. まとめ
本設計はMySQL（InnoDB）を前提とし、
外部キー制約・トランザクション・JOIN・副問い合わせを用いた
ポケモン対戦支援データベースである。
