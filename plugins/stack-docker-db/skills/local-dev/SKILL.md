---
name: local-dev
description: >-
  Use this skill when starting, stopping, resetting, or troubleshooting the local development environment,
  including Docker containers (database, emulation services, caches).
---

# Local Development Environment Skill

本スキルは、Docker コンテナ（データベース、キャッシュ、エミュレータ等）を用いたローカル開発環境の起動・停止・リセットおよび接続確認を動的探索に基づいて行うための手順書です。

---

## 運用原則: 動的コマンド特定 (Dynamic Discovery)

特定のディレクトリパスやスクリプトをハードコードせず、プロジェクト内の定義ファイルを動的に確認して操作コマンドを特定します。

1. **タスクランナーの確認**:
   - ルートの `Makefile`（`make help` またはターゲット定義）を確認。
   - `package.json`（`scripts` 内の `docker`, `db`, `dev` 関連）を確認。
2. **コンテナ定義の確認**:
   - `docker-compose.yml` または `compose.yaml` のサービス定義を確認。
3. **環境変数ファイルの確認**:
   - `.env.example` の存在を確認し、必要に応じて `.env` を準備。

---

## 主要な操作手順

### 1. タスクランナーおよび設定の探索

実行前に利用可能なコマンドを特定します:

```bash
# Makefile のターゲット一覧を確認
make help 2>/dev/null || grep -E '^[a-zA-Z0-9_-]+:' Makefile

# package.json の scripts を確認
cat package.json | grep -A 20 '"scripts"'

# Docker Compose 設定の確認
docker compose config --services
```

---

### 2. 環境の起動・初期化

特定したタスクコマンド、または標準 Docker Compose コマンドを実行します:

```bash
# 1. 環境変数ファイルの準備 (必要に応じて)
[ -f .env.example ] && [ ! -f .env ] && cp .env.example .env

# 2. コンテナの起動
# タスクランナーがある場合: make up / pnpm run docker:up 等
# 直接実行する場合:
docker compose up -d
```

---

### 3. 稼働状態と疎通の確認

```bash
# コンテナのステータス確認
docker compose ps

# ログの確認
docker compose logs --tail=50
```

---

### 4. 環境のクリーン再初期化 (DB ボリュームリセット)

マイグレーション検証やデータリセット時:

```bash
# タスクランナーがある場合: make reset / pnpm run db:reset 等
# 直接実行する場合:
docker compose down -v && docker compose up -d
```
