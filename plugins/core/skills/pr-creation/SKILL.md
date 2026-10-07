---
name: pr-creation
description: >-
  Use this skill when preparing to open a Pull Request (PR), completing a feature/task branch,
  or when the user requests to create a PR on GitHub.
---

# Pull Request (PR) Creation Skill

本スキルは、動的探索による必須事前検証をパスし、規約に従って Draft Pull Request を作成・提出するための手順書です。

---

## 実行プロシージャ

### 1. PR 作成前の動的探索と必須事前検証
PR を作成する前に、プロジェクトの定義ファイル（`package.json`, `Makefile`, `Taskfile` 等）を動的に確認し、フォーマット、リント、テスト、ビルド検証を実行してすべてエラーなくパスしていることを確認します。

```bash
# 1. 利用可能な検証タスクを動的に確認
if [ -f Makefile ]; then
  make help 2>/dev/null || grep -E '^[a-zA-Z_-]+:' Makefile
elif [ -f package.json ]; then
  cat package.json | grep -E '"(format|lint|test|build|check)'
fi

# 2. プロジェクトの規約に応じた検証コマンドの実行 (例: pnpm / npm / yarn / make)
# - フォーマット検証 (例: pnpm run format:check または make format)
# - 静的解析・リント検証 (例: pnpm run lint または make lint)
# - テスト実行 (例: pnpm run test または make test)
# - ビルド検証 (例: pnpm run build または make build)
```

> **注意**: ビルドやテストに失敗している状態では PR を作成してはいけません。不整合があれば修正を行ってください。

---

### 2. ブランチ名の確認・検証
現在の作業ブランチ名がリポジトリのブランチ命名規則に適合しているか確認します。

```bash
git branch --show-current
```

- **形式**: `<type>/<issue-number>/<short-description>`
  - 例: `feature/12/user-authentication`, `fix/34/db-pool-leak`
- 命名規則に沿っていない場合は、`git branch -m <correct-branch-name>` で修正してください。

---

### 3. PR の作成方針 (Draft PR 原則)
- **原則 Draft 作成**:
  - PR は原則として **Draft（下書き）** 状態で作成します。
  - CI チェックの通過および自身による差分セルフレビュー（不要なログやデバッグコードの混入がないか等）を確認した後に Ready for Review へ移行します。

---

### 4. PR タイトルおよび本文の規約

#### 4.1 PR タイトル
- Conventional Commits プレフィックス（`feat:`, `fix:`, `docs:` など）を付与し、変更内容を簡潔に表現します。
  - 例: `feat(auth): add jwt refresh token rotation mechanism`
  - 例: `fix(database): resolve connection pool leak on shutdown`

#### 4.2 PR 本文
PR 本文には以下のセクションを含めます:

```markdown
## Overview
<!-- 変更の背景と目的を簡潔に記載 -->

## Changes
<!-- 主な変更点を箇条書きで記載 -->
- 
- 

## Verification
<!-- ローカルで実施した検証手順と結果 -->
- [x] フォーマット・リント検証
- [x] テスト実行
- [x] ビルド検証

## Issues & References
<!-- 自動クローズ対象の Issue 番号 -->
resolves #<issue-number>
```

---

### 5. PR の作成コマンド実行

GitHub CLI (`gh`) を使用して Draft PR を作成します:

```bash
# リモートへプッシュ
git push -u origin HEAD

# Draft PR の作成
gh pr create --draft --title "<type>(<scope>): <title>" --body "<PR本文>"
```

---

### 6. 結果の確認と報告
1. コマンドが出力した PR の URL を取得します。
2. ユーザーに対し、作成した PR の URL、タイトル、およびサマリーを報告します。
