---
name: frontend-development
description: >-
  Use this skill when developing UI components, custom hooks, context providers,
  or testing and building frontend web applications.
---

# Frontend Web Application Development Skill

本スキルは、React / Web フロントエンドアプリケーションにおいて、UI コンポーネント、カスタム Hook、Context/Provider の作成および動的検証を行うための手順書です。

---

## 開発プロシージャ

### Step 1: カスタム Hook によるロジック分離

状態管理や外部 API とのやり取りは、コンポーネントから分離してカスタム Hook に記述します。

```typescript
// hooks/useToggle.ts
import { useState, useCallback } from 'react';

export function useToggle(initialState = false): [boolean, () => void] {
  const [state, setState] = useState(initialState);
  const toggle = useCallback(() => setState((prev) => !prev), []);
  return [state, toggle];
}
```

---

### Step 2: UI コンポーネントの実装

PascalCase でコンポーネントファイルを作成し、型安全な Props を定義します。

```tsx
// components/ModalDialog.tsx
import React, { ReactNode } from 'react';

interface ModalDialogProps {
  isOpen: boolean;
  onClose: () => void;
  title: string;
  children: ReactNode;
}

export function ModalDialog({ isOpen, onClose, title, children }: ModalDialogProps) {
  if (!isOpen) return null;

  return (
    <div role="dialog" aria-modal="true" aria-labelledby="modal-title">
      <h2 id="modal-title">{title}</h2>
      <div>{children}</div>
      <button type="button" onClick={onClose}>閉じる</button>
    </div>
  );
}
```

---

### Step 3: Context & Provider の安全な公開

Context を直接 export せず、カスタム Hook 経由で取得させることで、Provider 外での誤用を早期検知します。

```tsx
// contexts/ThemeContext.tsx
import React, { createContext, useContext, useState, ReactNode } from 'react';

type Theme = 'light' | 'dark';

interface ThemeContextType {
  theme: Theme;
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export function ThemeProvider({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState<Theme>('light');
  const toggleTheme = () => setTheme((prev) => (prev === 'light' ? 'dark' : 'light'));

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme(): ThemeContextType {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}
```

---

### Step 4: 動的探索による検証プロシージャ

対象フロントエンドパッケージの `package.json`（`scripts`）を動的に確認し、型チェック、リント、テスト、ビルドを実行します。

```bash
# 対象パッケージの scripts を確認
cat apps/<frontend-app>/package.json | grep -A 10 '"scripts"'

# 型検査
pnpm --filter <app-name> run typecheck  # または tsc --noEmit

# リント検証
pnpm --filter <app-name> run lint

# 単体・コンポーネントテスト
pnpm --filter <app-name> run test

# プロダクションビルド検証
pnpm --filter <app-name> run build
```
