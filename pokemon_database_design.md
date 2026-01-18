Markdown

# ポケモン対戦支援システム データベース設計書

## 1. システム構成図 (System Architecture)

本システムはMySQLをデータベースとして採用し、PythonやNode.js等のアプリケーションサーバーを介してユーザーと対話する構成とする。

```mermaid
graph TD
    User[ユーザー (トレーナー)] -->|ポケモン登録 / 対戦相手入力| App[アプリケーション]
    App -->|クエリ実行 (SQL)| DB[(MySQL Database)]
    DB -->|検索結果 (最適なポケモン)| App
    App -->|結果表示| User
2. データベース設計 (ER図 & 正規化)
データの整合性を保つため、第3正規形 (3NF) まで正規化を行っている。

正規化の適用
第1正規形: 繰り返し項目の排除。

第2正規形: ポケモンの種族データ（名称、種族値）を my_pokemons テーブルから分離し、species テーブルとして独立させる。

第3正規形: 計算可能な項目や導出項目（タイプ相性など）をマスタデータ type_chart として分離。

ER図 (Entity Relationship Diagram)
コード スニペット

erDiagram
    SPECIES ||--o{ MY_POKEMONS : "is instance of"
    SPECIES }|--|| TYPES : "has type 1"
    SPECIES }|--|| TYPES : "has type 2"
    TYPES ||--o{ TYPE_CHART : "attacker"
    TYPES ||--o{ TYPE_CHART : "defender"

    %% ポケモン図鑑データ（マスタ）
    SPECIES {
        int species_id PK "図鑑番号"
        string name "ポケモン名"
        int base_speed "種族値: すばやさ"
        string type1_id FK "タイプ1"
        string type2_id FK "タイプ2 (NULL可)"
    }

    %% 手持ちポケモン（トランザクション）
    MY_POKEMONS {
        int id PK "管理ID"
        int species_id FK "種族ID"
        string nickname "ニックネーム"
        int level "レベル"
        int individual_speed "実数値: すばやさ"
        datetime created_at "登録日"
    }

    %% タイプマスタ
    TYPES {
        string type_id PK "タイプ名 (例: Fire)"
    }

    %% タイプ相性表
    TYPE_CHART {
        int id PK
        string attacker_type_id FK "攻撃側タイプ"
        string defender_type_id FK "防御側タイプ"
        float multiplier "倍率 (2.0, 0.5, 1.0, 0.0)"
    }
3. テーブル定義 (DDL概要)
各テーブルには適切な主キー(PK)および外部キー(FK)制約を設定する。

species (種族マスタ)
PRIMARY KEY: species_id

Constraint: name は UNIQUE制約とする。

my_pokemons (手持ちポケモン)
PRIMARY KEY: id (AUTO_INCREMENT)

FOREIGN KEY: species_id は species(species_id) を参照する。

type_chart (タイプ相性)
Check Constraint: multiplier カラムには有効な倍率値のみ許可するよう設定を推奨。

4. 機能別 SQL設計 & ロジック
① 手持ちポケモンの登録 (Transaction)
データの不整合を防ぐため、登録処理はトランザクションを用いて実行する。

SQL

START TRANSACTION;

-- 例: レベル50のピカチュウ(ID:25)を登録する場合
INSERT INTO my_pokemons (species_id, nickname, level, individual_speed, created_at)
VALUES (25, 'マイピカ', 50, 150, NOW());

COMMIT;
② 最適な手持ちポケモンの選出 (Join & Subquery)
以下の条件に基づき、最適なポケモンを抽出する。

攻撃相性が有利であること（タイプ相性倍率が2.0以上）

相手よりすばやさが高いこと

前提: 対戦相手のポケモンが「タイプ: Fire」「すばやさ: 160」である場合。

SQL

-- 相手のタイプ(Fire)に対して有利、かつ相手より速い自分のポケモンを抽出
SELECT 
    mp.nickname,
    s.name AS species_name,
    mp.individual_speed,
    tc.multiplier AS type_advantage
FROM 
    my_pokemons mp
JOIN 
    species s ON mp.species_id = s.species_id
JOIN 
    -- 自身のタイプ1で攻撃する場合の相性を結合
    type_chart tc ON tc.attacker_type_id = s.type1_id
WHERE 
    tc.defender_type_id = 'Fire' -- 敵のタイプ
    AND tc.multiplier >= 2.0     -- 効果抜群以上
    AND mp.individual_speed > 160 -- 相手より速い
ORDER BY 
    tc.multiplier DESC,          -- 倍率が高い順
    mp.individual_speed DESC;    -- さらに速い順
副文 (Subquery) の活用例
平均レベル以上のポケモンのみを検索対象とする場合などに副文を使用する。

SQL

SELECT * FROM my_pokemons 
WHERE level > (SELECT AVG(level) FROM my_pokemons); -- 平均レベルより高い個体のみ抽出
5. 今後の拡張性について
現行の仕様は基本的な構成であるが、今後の拡張として以下が考えられる。

技（Moves）テーブルの追加: ポケモン自身のタイプではなく、習得している「技のタイプ」に基づいた相性計算の実装。

特性（Abilities）の実装: 「ふゆう」等の特性によるタイプ相性の無効化・変化ロジックの追加。
