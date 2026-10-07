---
name: pnpm-workspace
description: >-
  Use this skill when managing packages, dependencies, build caches, or running workspace commands
  in a pnpm monorepo.
---

# pnpm Workspace Management Skill

本スキルは、pnpm workspace モノレポ環境において、パッケージの追加・依存管理・ビルド・テスト検証を動的に実行するための手順書です。

---

## 実行プロシージャ

### Step 1: ワークスペース構成の動的探索

固定のパスを前提とせず、まずはリポジトリルートの設定ファイルを確認してパッケージ構成を把握します。

```bash
# ワークスペース定義の確認
cat pnpm-workspace.yaml

# 参加しているパッケージ一覧の確認
pnpm ls -r --depth -1
```

---

## 主要な操作手順

### 1. 外部依存パッケージの追加
特定パッケージに外部ライブラリを追加します:

```bash
# 対象パッケージに依存を追加
pnpm --filter <package-name> add <dependency-name>

# 開発依存 (devDependencies) を追加
pnpm --filter <package-name> add -D <dependency-name>

# ルートワークスペースに追加 (ツールチェーン・共通設定等)
pnpm add -Dw <dependency-name>
```

### 2. モノレポ内共有パッケージの参照リンク (`workspace:*`)
モノレポ内の他パッケージを依存関係として追加します:

```bash
# workspace:* プロトコルで内部パッケージをリンク
pnpm --filter <app-or-package> add <shared-package>@workspace:*
```

### 3. 動的タスク実行 (ビルド・型チェック・テスト)
特定のパッケージまたはワークスペース全体に対してタスクを実行します。事前に各パッケージの `package.json`（`scripts`）を確認してください。

```bash
# 対象パッケージの scripts を確認
cat apps/<app-name>/package.json | grep -A 10 '"scripts"'

# 特定パッケージのタスク実行
pnpm --filter <package-name> run <script-name>

# ワークスペース全体のタスク並行実行
pnpm -r run <script-name>
```

### 4. 依存関係の整合性・重複チェック
依存キャッシュの不整合や重複を解消する場合:

```bash
# 依存関係のクリーンインストール
pnpm install --frozen-lockfile

# 重複した依存バージョンの整理
pnpm dedupe
```
