---
name: tdd
description: >-
  Use this skill when developing new features, implementing domain/application logic,
  or fixing bugs following the Test Driven Development (TDD) cycle and coordinating test-writer and implementer subagents.
---

# Test Driven Development (TDD) Skill

本スキルは、厳格な TDD（Red-Green-Refactor サイクル）を遂行し、必要に応じて `test-writer`（テスト作成）と `implementer`（実装）の 2 つの専門サブエージェントを連携させて高品質なコードを開発するための手順書です。

---

## TDD 実行サイクル

```text
[要件定義・仕様確認]
       │
       ▼
[Phase 1: Red]   ---> test-writer サブエージェントがテスト先行作成 (*.spec.ts)
       │               テスト実行し、意図通りの理由で FAIL することを確認
       ▼
[Phase 2: Green]  ---> implementer サブエージェントが最小限の実装コードを作成
       │               テスト実行し、全件 PASS することを確認
       ▼
[Phase 3: Refactor] -> 重複排除、型定義洗練、動的検証の実行 (テストが PASS を維持)
```

---

## 実行プロシージャ

### 準備: プロジェクトのテストコマンド動的探索

固定コマンドを前提とせず、プロジェクトルートまたは対象パッケージの `package.json`（`scripts`）や `Makefile` を動的に確認してテスト実行コマンドを特定します。

```bash
# 対象パッケージのテストスクリプト確認例
cat package.json | grep -E '"(test|jest|vitest)'
```

---

### Phase 1: Red (テスト先行作成)

1. **テスト配置の決定**:
   - 単体テスト: 実装予定ファイルと同階層の `<target>.spec.ts`
   - E2E / 統合テスト: アプリケーションまたはパッケージの `test/` 配下
2. **サブエージェント `test-writer` の起動**:
   - `subagents/test-writer.md` の役割定義をプロンプトに含め、テスト作成を委譲します（Antigravity の場合は `invoke_subagent` 等を活用）。
   - 要求仕様に対する正常系・異常系・境界値テストを記述させます。
3. **テストの失敗確認 (Red)**:
   - 特定したテストコマンドにテストファイルパスを渡して実行します:
     ```bash
     # 例: パッケージマネージャやテストランナーに応じた実行
     # pnpm run test -- <path/to/spec.ts>
     # pnpm --filter <package-name> test -- <path/to/spec.ts>
     ```
   - 実装が存在しないことによる未定義・アサーション不一致で**確実に FAIL すること**を確認します。

---

### Phase 2: Green (最小限の実装)

1. **サブエージェント `implementer` の起動**:
   - `subagents/implementer.md` の役割定義をプロンプトに含め、実装を委譲します。
   - テストをパスさせるための最小限のコードのみを実装させます（YAGNI 原則の遵守）。
2. **テストの成功確認 (Green)**:
   - 対象テストを再実行し、**すべて PASS すること**を確認します。

---

### Phase 3: Refactor & Verification

1. **リファクタリング**:
   - コードの可読性向上、設計パターンの適用、エラーハンドリングの共通化を行います。
   - リファクタリング中も頻繁に対象テストを実行し、PASS を維持します。
2. **品質検証**:
   - プロジェクト定義のリンター・フォーマッターを実行します。
   - パッケージ全体のテストおよびビルドを実行してリグレッションがないことを確認します。
