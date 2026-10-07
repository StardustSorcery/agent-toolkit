# Agent Core Guidelines

本ドキュメントは、プロジェクトで作業するすべての AI エージェント（Antigravity、Claude Code 等）に対する共通行動指針および作業規約を定めます。

---

## 1. エージェントのコア行動原則

### 1.1 動的探索の徹底 (Dynamic Discovery)
- ワークスペース内のパッケージ構成やビルド設定は動的に変化します。
- 特定のディレクトリパスやビルドコマンドを固定的に前提とせず、ルートの定義ファイル（`package.json`, `pnpm-workspace.yaml`, `Makefile`, `Taskfile` 等）を動的に確認して実行コマンドを判断してください。
- 実行可能なタスクの特定には、`make help` や `pnpm run` などの一覧コマンドを活用します。

### 1.2 ドキュメント正本の尊重 (Single Source of Truth)
- 仕様・アーキテクチャ・コーディング規約の正本は常に `docs/` 配下で管理されます。
- 設計や機能追加・修正に着手する前に、必ず関連ドキュメントを確認してください。
- 実装変更によって仕様や設計方針が変化した場合は、コードと同時に対応するドキュメントも更新してください。

### 1.3 低コンテキスト探索の遵守 (Low-Context Discovery)
- `docs/` 配下を探索する際は全ドキュメントを無差別に読み込まず、Open Knowledge Format (OKF) の frontmatter（`type`, `title`, `description`, `tags`）を活用してピンポイントで必要なファイルのみを精読してください。
- まずファイル一覧（`find` やディレクトリ走査）を取得し、候補ファイルの先頭行（frontmatter）を確認してから対象セクションを読み込みます。

### 1.4 テスト駆動開発 (TDD) の推進
- 機能実装やバグ修正時は、テストを先行作成する **Red-Green-Refactor サイクル** を厳格に遵守してください。
- テスト作成（`test-writer`）と実装（`implementer`）の責務を分離して進めることを推奨します。
- テストが意図通りの理由で失敗（Red）したことを確認してから、テストをパスさせる最小限の実装（Green）を行い、その後にリファクタリングを実施します。

---

## 2. ブランチ戦略およびコミット規約

### 2.1 ブランチ戦略 (GitHub Flow ベース)
GitHub Flow をベースとし、Issue に紐づく作業ブランチを作成して開発を行います。

#### ブランチ命名規則
作業ブランチは以下の 3 階層構造とします：

```text
<type>/<issue-number>/<short-description>
```

- `<type>`: 変更の種類を表すプレフィックス
  - `feature`: 新機能の追加・仕様変更
  - `fix`: バグ修正
  - `docs`: ドキュメントの追加・修正
  - `refactor`: リファクタリング（機能変更・バグ修正を含まないコード整理）
  - `test`: テストコードの追加・改善
  - `chore`: ビルド設定、ツールチェーン、依存関係の更新など
- `<issue-number>`: 対象となる Issue 番号（数字のみ）
- `<short-description>`: 変更内容を端的に表す英数字とハイフン（小文字）

例: `feature/12/user-authentication`, `fix/45/connection-timeout`

#### ベースブランチ (マージ先)
- Pull Request のマージ先は、プロジェクトの運用方針に従いマイルストーンブランチまたは `main` ブランチを指定します。

---

### 2.2 コミットメッセージ規約 (Conventional Commits)
コミットメッセージは Conventional Commits 形式に準拠します。

```text
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

#### `<type>` の種類
- `feat`: 新機能追加
- `fix`: バグ修正
- `docs`: ドキュメントの変更
- `style`: コードの動作に影響しないフォーマット変更
- `refactor`: リファクタリング
- `perf`: パフォーマンス改善
- `test`: テストの追加・修正
- `chore`: ビルドプロセスや補助ツールの変更

#### コミット前検証
コミットを作成する前に、ローカル環境でコードフォーマット、リント、テストの整合性を必ず確認してください。
