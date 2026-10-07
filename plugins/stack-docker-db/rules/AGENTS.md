# Database & DDL Design Guidelines

本ドキュメントは、データベーススキーマ（PostgreSQL 等）の DDL 設計、マイグレーション管理、シードデータ作成においてエージェントが遵守すべき設計原則および規約を定めます。

---

## 1. ファイル配置と連番命名規則

マイグレーションファイルは、実行順序を一意に保証するため **6 桁ゼロ埋めの連番プレフィックス** を付与します。

```text
schema/
├── migrations/ (または postgres/ 等)
│   ├── 000001_create_initial_tables.sql
│   ├── 000002_add_user_roles.sql
│   └── 000003_add_audit_logs.sql
└── seeds/
    └── 000001_initial_seed_data.sql
```

- 新規マイグレーションを作成する際は、既存の最大連番を確認し、それに 1 を加えた番号（例: `000004_<description>.sql`）を付与します。
- マイグレーションファイルはソート順（辞書順）で順次適用されることを前提とします。

---

## 2. DDL 設計規則

### 2.1 命名規則
- **テーブル名**: 複数形スネークケース（例: `users`, `accounts`, `order_items`）
- **カラム名**: 単数形スネークケース（例: `user_id`, `email`, `created_at`）
- **インデックス名**: `idx_<テーブル名>_<カラム名>`（例: `idx_order_items_order_id`）
- **ユニーク制約名**: `uq_<テーブル名>_<カラム名>`（例: `uq_users_email`）

### 2.2 主キー・外部キー設計
- **主キー**: 原則として UUID を採用します。
  ```sql
  id UUID PRIMARY KEY DEFAULT gen_random_uuid()
  ```
  ※ パフォーマンスやシーケンシャル性が求められる場合は `BIGINT GENERATED ALWAYS AS IDENTITY` を検討。
- **外部キー**: `REFERENCES <table_name>(id)` を明示し、`ON DELETE CASCADE` や `ON DELETE RESTRICT` などの削除ポリシーを必ず指定します。
- **外部キーインデックス**: 結合性能を担保するため、外部キーカラムには必ずインデックスを作成してください:
  ```sql
  CREATE INDEX IF NOT EXISTS idx_<table_name>_<fk_column> ON <table_name>(<fk_column>);
  ```

### 2.3 日時カラム
- 日時を扱うカラムにはタイムゾーン付きタイムスタンプ型を採用します:
  ```sql
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
  ```

### 2.4 冪等性の担保
- 適用スクリプトには可能な限り `CREATE TABLE IF NOT EXISTS` や `CREATE INDEX IF NOT EXISTS` を使用し、安全な再実行を可能にします。

---

## 3. シードデータの作成原則

- テスト・ローカル検証用データは、外部キー制約の依存順（親テーブルから子テーブルへ）に従って作成します。
- 本番機への誤投入を防ぐため、シードデータはマイグレーションファイルと分離された専用ディレクトリ（`seeds/` 等）で管理します。

---

## 4. スキーマ検証原則

- マイグレーションファイルや DDL の追加・変更時は、クリーンな環境（DB ボリューム初期化）で再現性高くスキーマおよびシードデータが適用されることを必ず検証します。
- 外部キー制約の不整合や構文エラーを早期検出するため、CI およびローカルでの初期化・適用検証を前提とした冪等な記述を徹底してください。
