---
name: doc-authoring
description: >-
  Use this skill when creating or updating documentation, Architecture Decision Records (ADRs),
  operational guides, or rules in the docs/ directory.
---

# Documentation Authoring Skill

本スキルは、リポジトリにおいて Open Knowledge Format (OKF v0.2) に準拠した高品質な技術ドキュメントを作成・更新するための手順書です。

---

## 1. ドキュメント配置体系

すべてのドキュメントは、そのライフサイクルと適用範囲に応じて適切なディレクトリに配置します。

```text
docs/
├── vX-X-X/                              # マイルストーン・バージョン固有のドキュメント (例: v1-0-0/)
│   └── adr-XXXX-<slug>.md               # アーキテクチャ決定記録 (ADR)・設計書
├── rules/                               # プロジェクト共通の運用・開発規約 (普遍的)
│   └── <rule-name>.md                   # ブランチ戦略、PR 規約、アーキテクチャ規約等
└── guides/                              # 開発環境・ツール運用のハウツーガイド
    └── <guide-name>.md                  # 環境構築手順、運用手順等
```

---

## 2. Open Knowledge Format (OKF v0.2) 準拠 frontmatter

`docs/` 配下に配置するすべての Markdown ドキュメントは、ファイルの先頭に YAML frontmatter を記述します。

### 2.1 Frontmatter 仕様

```yaml
---
type: <Type name>             # 【必須】コンセプトの種類
title: <Display Title>        # 【必須】ドキュメントの正式名称
description: <One-line summary> # 【推奨】1行での要約 (探索・要約用)
tags:                         # 【推奨】横断的分類タグ
  - <tag1>
  - <tag2>
---
```

### 2.2 主要な `type` の定義
- `ADR`: アーキテクチャ決定記録 (Architecture Decision Record)
- `Rule`: 開発・運用において遵守すべき規約・制約
- `Guide`: セットアップ手順や操作方法の解説ガイド
- `Specification`: API、データモデル、外部インターフェースの仕様書
- `Reference`: リファレンス資料・チートシート

---

## 3. ドキュメント記述ガイドライン

1. **構造化の徹底**:
   - 見出し階層（`#`, `##`, `###`）を正しく使い、箇条書きや比較テーブルを活用して視認性を高めます。
2. **Mermaid ダイアグラムの活用**:
   - アーキテクチャ構成図、シーケンス、状態遷移、依存関係はコードブロック形式の Mermaid（`mermaid`）で記述します。
3. **相対パスリンク**:
   - 他ドキュメントへの参照は相対パスを用いた Markdown リンク（例: `[アーキテクチャ規約](../rules/architecture.md)`）を使用します。
4. **自己完結性と最新性の担保**:
   - 将来のメンテナンスを見据え、前提条件や背景となる文脈を明確に記録します。
