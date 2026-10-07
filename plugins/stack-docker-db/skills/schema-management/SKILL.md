---
name: schema-management
description: >-
  Use this skill when adding or modifying database DDL tables, migrations,
  or local seed data in schema and container directories.
---

# Schema Management Skill

本スキルは、データベースのテーブル定義（DDL）およびマイグレーションファイルの追加・更新・ローカル検証を動的探索に基づいて行うための手順書です。

---

## 実行プロシージャ

### Step 1: 既存マイグレーション番号の確認

スキーマ管理ディレクトリ内の既存ファイル一覧を確認し、末尾の連番番号（6 桁）を特定します。

```bash
# マイグレーションディレクトリの特定と一覧確認
find . -type d \( -name "migrations" -o -name "schema" \) -maxdepth 3
ls -1 <migrations-dir>/
```

新しく作成するファイルは、既存の最大連番に 1 を加えた番号とします（例: 最大が `000003_...` なら次は `000004_<description>.sql`）。

---

### Step 2: DDL ファイルの作成

新規マイグレーションファイル（例: `00000X_<description>.sql`）を作成します。

```sql
-- DDL 記述例
CREATE TABLE IF NOT EXISTS users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) NOT NULL UNIQUE,
    name VARCHAR(255) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX IF NOT EXISTS idx_users_email ON users (email);
```

---

### Step 3: シードデータの追加

新設したテーブルや追加した必須カラムに対して、ローカル開発・テスト用のデータをシード用ディレクトリ（`schema/seeds/` 等）に作成・追記します。

---

### Step 4: 動的探索によるスキーマ初期化検証

定義完了後、プロジェクトのタスクランナー（`Makefile` や `package.json` 等）または Docker Compose 設定を確認し、適切なリセットコマンドで DB を再初期化して DDL とシードがエラーなく実行されることを検証します。

```bash
# 1. プロジェクトのリセット・マイグレーションコマンドを動的に確認
if [ -f Makefile ]; then
  make help 2>/dev/null || grep -E '^[a-zA-Z_-]+:' Makefile
elif [ -f package.json ]; then
  cat package.json | grep -E '"(db|migrate|reset)'
fi

# 2. 特定したコマンド、または標準 Docker Compose コマンドで再初期化
# 例: make reset または docker compose down -v && docker compose up -d

# 3. 疎通・テーブル反映の確認
# 例: make query または psql / 各種クライアントで接続確認
```
