# Frontend Web Apps Architecture Guidelines

本ドキュメントは、React、Next.js、Vite 等を用いたフロントエンド Web アプリケーション（SPA / SSR / PWA）において、コンポーネント設計、カスタム Hook、状態管理、命名規約、および実行環境境界の原則を定めます。

---

## 1. コンポーネント設計原則

1. **関心の分離と単一責任 (Single Responsibility)**:
   - UI 表示の責務（Presentational）と状態管理・データ取得の責務（Container / Custom Hook）を適切に分離します。
2. **Prop Drilling の防止**:
   - 複数階層に渡る Props のバケツリレーを避け、React Context、状態管理ライブラリ、または Composition パターン（children 経由）を活用します。
3. **副作用の局所化**:
   - `useEffect` の乱用を避け、イベントハンドラやカスタム Hook 内で副作用を完結させます。

---

## 2. ファイル命名およびサフィックス規約

| 分類 | 命名形式 | ファイル名例 | 備考 |
| :--- | :--- | :--- | :--- |
| **Component** | `PascalCase.tsx` | `UserProfileCard.tsx`, `Header.tsx` | UI をレンダリングするコンポーネント |
| **Custom Hook** | `use<CamelCase>.ts` | `useAuth.ts`, `useWindowSize.ts` | ロジックや状態を持つカスタム Hook |
| **Context / Provider** | `<PascalCase>Context.tsx` | `AuthContext.tsx`, `ThemeProvider.tsx` | グローバル状態の提供 |
| **Types** | `*.types.ts` | `user.types.ts` | 型定義・インターフェース専用 |
| **Styles** | `*.styles.ts` / `*.module.css` | `UserProfileCard.module.css` | コンポーネントスコープのスタイル |
| **Test** | `*.spec.tsx` / `*.test.tsx` | `UserProfileCard.spec.tsx` | 単体・統合テスト |

---

## 3. SPA / SSR / PWA 実行環境境界の原則

1. **SSR (Server-Side Rendering) 安全性**:
   - サーバーサイド実行時（Node.js ランタイム）には `window`, `document`, `localStorage` 等のブラウザ API は存在しません。
   - ブラウザ専用 API を参照する処理は、マウント検知（`useEffect`）後に行うか、環境チェック（`typeof window !== 'undefined'`）でガードします。
2. **サーバーキャッシュとクライアント状態の分離**:
   - サーバーから取得するデータ（Server Cache）と、UI 固有の揮発状態（Client State: モーダルの開閉、タブ選択等）を混同せず、適切なレイヤで保持します。
3. **PWA (Progressive Web Apps) キャッシュ整合性**:
   - Service Worker のキャッシュ戦略（Network First, Cache First 等）を意識し、更新検知とバージョン不整合を防ぐライフサイクル管理を行います。
