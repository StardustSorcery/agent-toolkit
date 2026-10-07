---
name: doc-search
description: >-
  Use this skill when searching for architecture decisions (ADRs), specifications, rules, or guides
  in docs/ while conserving LLM context window tokens.
---

# Documentation Search Skill (Low-Context Discovery)

本スキルは、`docs/` ディレクトリ配下に蓄積されたドキュメント群からコンテキストウィンドウの消費を最小限に抑えつつ目的の仕様や規約をピンポイントで特定・参照するための手順書です。

---

## 探索プロシージャ

### Step 1: 対象ファイルのリストアップ (ファイル名一覧の取得)

いきなりファイル内容を全文読み込まず、まずはファイル一覧を取得して全体像を把握します。

```bash
# docs/ 配下の Markdown ファイル一覧を取得
find docs -name "*.md" | sort
```

---

### Step 2: Frontmatter のピンポイント抽出 (低コンテキスト判定)

全ファイルを全文閲覧するとトークンを大量消費します。Open Knowledge Format (OKF) の frontmatter のみを確認して目的のファイルを絞り込みます。

#### 方法 A: キーワード検索 (`grep`)
探している概念（例: `database`, `auth`, `branch`, `api` 等）に関連するファイルを特定します:

```bash
grep -rn "title:.*auth" docs/
```

#### 方法 B: 先頭行のみの閲覧 (部分読み込み)
候補となったファイルの先頭 15〜25 行のみを部分閲覧します。

- `type`
- `title`
- `description`
- `tags`

これらを確認するだけで、ファイル全体の概要と探している情報が含まれているかを確実に判断できます。

---

### Step 3: 目的ドキュメントのピンポイント精読

目的に合致するファイルを特定したら、必要なセクションのみを読み込むか、該当箇所の行番号を確認して部分読み込みを行います。  
全体的な文脈把握が不可欠な場合のみ、ファイル全体を閲覧してください。
