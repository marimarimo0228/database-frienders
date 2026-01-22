# database-frienders
グループ名フレンダーズの課題リポジトリ

render用アドレス
https://pokemon-test-bd6t.onrender.com/

# ポケモン対戦支援システム (Pokémon Battle Support System)

## 概要 (Overview)
ポケモン対戦において、相手のポケモンを選択するだけで、ユーザーの手持ちの中から**「効果抜群（タイプ相性）」**かつ**「先制可能（素早さ）」**な最適なポケモンを瞬時に提示するWebアプリケーションです。
初心者が陥りがちな「判断の遅れ」や「知識不足」をデータ駆動で解決します。

## 主な機能 (Features)
* **対戦解析機能**: 敵ポケモンに対し、相性倍率と素早さを基準に有利な味方をランキング表示。
* **手持ち管理 (CRUD)**: ポケモンの登録、戦略メモの更新、削除。
* **カスタム登録**: 既存図鑑にないオリジナルポケモンの作成・登録。
* **可視化UI**: 「効果抜群（赤）」と「いまひとつ（青）」を色分け表示。
* **データ復旧 (DR)**: ワンクリックでデータベースを初期状態にリセットする機能。

## 技術スタック (Tech Stack)
本システムは **Web 3層構造 (Web 3-Tier Architecture)** を採用しています。

* **Presentation**: HTML5, CSS (Bootstrap 5), Jinja2
* **Application**: Python, Flask, Gunicorn
* **Database**: PostgreSQL (Render Managed)
* **Infrastructure**: Render (PaaS)

## データベース設計 (Database Design)
データの整合性と拡張性を担保するため、**第3正規形 (3NF)** まで正規化を行っています。

* **PokemonMaster**: ポケモン基本情報（マスタ）
* **UserPokemon**: ユーザーの手持ち情報（トランザクション）
* **Types**: タイプ定義
* **TypeEffectiveness**: タイプ相性倍率（交差テーブル）

## ローカルでの実行方法 (Local Setup)

```bash
# 1. リポジトリをクローン
git clone [https://github.com/](https://github.com/)[あなたのユーザー名]/[リポジトリ名].git

# 2. ディレクトリへ移動
cd [リポジトリ名]

# 3. 依存ライブラリのインストール
pip install -r requirements.txt

# 4. 環境変数の設定 (.envファイルを作成)
# DATABASE_URL=postgresql://user:password@localhost/dbname

# 5. アプリケーションの起動
python app.py
