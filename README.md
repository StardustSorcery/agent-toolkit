# agent-toolkit

AI コーディングエージェント（Antigravity、Claude Code 等）向けの再利用可能なマルチプラグイン・モノレポツールキットです。

本リポジトリは、開発ワークフロー、言語・スタック別の設計規約、およびインフラ管理機能を独立したプラグインとしてモジュール化して提供します。

---

## 提供プラグイン構成

| プラグイン名 | ディレクトリ | 概要 |
| :--- | :--- | :--- |
| **`agent-core`** | `plugins/core/` | 共通開発原則、ドキュメント規約、TDD サイクル、PR 作成等の基本ワークフロー |
| **`stack-pnpm-monorepo`** | `plugins/stack-pnpm-monorepo/` | pnpm workspace 管理、共有パッケージ境界、`workspace:*`、キャッシュ・ビルド分離 |
| **`stack-nestjs`** | `plugins/stack-nestjs/` | NestJS クリーンアーキテクチャ規約、レイヤ境界、依存性逆転、DI パターン、Interface/Impl 命名規約 |
| **`stack-web-apps`** | `plugins/stack-web-apps/` | フロントエンド Web アプリ設計、SPA/SSR/PWA、コンポーネント・Hook・状態管理 (Context/Provider) 規約 |
| **`stack-trpc`** | `plugins/stack-trpc/` | tRPC エンドツーエンド型安全、shared パッケージ型定義、satisfies ルーター実装、クライアント利用規約 |
| **`stack-docker-db`** | `plugins/stack-docker-db/` | Docker コンテナライフサイクル、データベース DDL 設計、マイグレーション連番規約、スキーマ管理 |

---

## プラグイン導入手順（標準: GitHub URL 直接インストール）

Antigravity および Claude Code ともに、**ローカルへの clone や Git submodule は不要**です。
GitHub のリポジトリ URL を直接指定することで、エージェントがリモートから自動取得してインストールします。

---

### 1. Antigravity での導入手順

Antigravity では、チャットセッション内または CLI コマンド (`agy`) からリポジトリ URL を直接指定してインストールできます。

#### リポジトリ URL による一括インストール
リポジトリ URL を指定すると、含まれるプラグイン群をまとめてインストールできます。

- **チャットセッション内**:
  ```text
  /plugin install https://github.com/StardustSorcery/agent-toolkit
  ```
- **CLI ターミナル (`agy`)**:
  ```bash
  agy plugin install https://github.com/StardustSorcery/agent-toolkit
  ```

#### 特定プラグインのみの選択的インストール（GitHub tree URL 指定）
GitHub のツリー URL（パス付き URL）を指定することで、プロジェクトに必要なプラグインだけを選択してインストールできます。

- **コアワークフローのみ導入する場合**:
  ```bash
  agy plugin install https://github.com/StardustSorcery/agent-toolkit/tree/main/plugins/core
  ```
- **特定スタックのみ導入する場合（例: NestJS）**:
  ```bash
  agy plugin install https://github.com/StardustSorcery/agent-toolkit/tree/main/plugins/stack-nestjs
  ```

#### プラグインの管理（確認・有効化・無効化）
インストール済みプラグインの一覧確認や、有効化・無効化の切り替えは以下のコマンドで行います:

```bash
# インストール済みプラグインの一覧表示
agy plugin list

# プラグインの有効化 / 無効化
agy plugin enable <name>
agy plugin disable <name>
```

---

### 2. Claude Code での導入手順

Claude Code では、GitHub リポジトリ URL をマーケットプレイスとして直接登録し、必要なプラグインを選択してインストールします（ローカルへの clone は不要です）。

#### 手順 1: マーケットプレイスの直接登録
リポジトリの URL をマーケットプレイスとして追加します:

```bash
/plugin marketplace add https://github.com/StardustSorcery/agent-toolkit
```

#### 手順 2: 必要なプラグインの選択インストール
マーケットプレイス名（`@agent-toolkit`）を指定して、必要なプラグインをインストールします:

```bash
# コア機能
/plugin install agent-core@agent-toolkit

# スタック別プラグイン（プロジェクトに合わせて選択）
/plugin install stack-pnpm-monorepo@agent-toolkit
/plugin install stack-nestjs@agent-toolkit
/plugin install stack-web-apps@agent-toolkit
/plugin install stack-trpc@agent-toolkit
/plugin install stack-docker-db@agent-toolkit
```

---

## 代替手順（高度な運用 / オフライン環境向け）

チーム開発でプラグイン構成をバージョン固定してリポジトリにコミット管理したい場合や、オフライン環境・閉域網等で運用する場合は、Git Submodule や手動配置による導入も可能です。

### 方法 A: Git Submodule + プロジェクト設定 (`.agents/plugins.json`)
プロジェクト内でリポジトリを Submodule として保持し、`.agents/plugins.json` で読み込むプラグインを明示的に指定します。

1. サブモジュールの追加:
   ```bash
   git submodule add https://github.com/StardustSorcery/agent-toolkit tools/agent-toolkit
   ```
2. `.agents/plugins.json` の作成（`templates/plugins.json` をコピーまたは作成）:
   ```json
   {
     "entries": [
       {
         "path": "tools/agent-toolkit/plugins",
         "include_only": [
           "core",
           "stack-pnpm-monorepo",
           "stack-nestjs"
         ]
       }
     ]
   }
   ```

### 方法 B: ワークスペース `.agents/plugins/` への手動配置（シンボリックリンク）
プロジェクトルートの `.agents/plugins/` 配下にプラグインディレクトリを配置またはシンボリックリンクを作成すると、エージェントにより自動認識されます:

```bash
mkdir -p .agents/plugins
ln -s /path/to/agent-toolkit/plugins/core .agents/plugins/agent-core
ln -s /path/to/agent-toolkit/plugins/stack-nestjs .agents/plugins/stack-nestjs
```

Claude Code で Submodule 運用を行う場合は、`.claude/settings.json` の `extraKnownMarketplaces` にローカルパスを定義することで自動認識させることも可能です:

```json
{
  "extraKnownMarketplaces": [
    {
      "name": "agent-toolkit",
      "path": "./tools/agent-toolkit"
    }
  ]
}
```