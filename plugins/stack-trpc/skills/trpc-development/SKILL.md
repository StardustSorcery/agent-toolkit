---
name: trpc-development
description: >-
  Use this skill when defining tRPC routers, shared API types, client generation,
  or testing and building end-to-end type-safe APIs.
---

# tRPC Development Skill

本スキルは、tRPC を用いたエンドツーエンド型安全な API 開発において、バックエンドルーターの作成、shared 型定義のエクスポート、フロントエンドクライアントの利用および動的検証を行うための手順書です。

---

## 開発プロシージャ

### Step 1: バリデーション付きバックエンド Router の作成

Zod スキーマで入力を検証し、プロシージャを定義します。

```typescript
// server/src/routers/user.router.ts
import { initTRPC } from '@trpc/server';
import { z } from 'zod';

const t = initTRPC.create();

export const userRouter = t.router({
  getUserById: t.procedure
    .input(z.object({ id: z.string().uuid() }))
    .query(async ({ input }) => {
      return { id: input.id, name: 'Sample User' };
    }),

  updateUserName: t.procedure
    .input(z.object({ id: z.string().uuid(), name: z.string().min(1) }))
    .mutation(async ({ input }) => {
      return { success: true, id: input.id, name: input.name };
    }),
});
```

---

### Step 2: ルーター型の公開 (Shared / Type Only)

実装コードを含めず、ルーターの型定義のみを公開します。

```typescript
// server/src/routers/app.router.ts
import { initTRPC } from '@trpc/server';
import { userRouter } from './user.router';

const t = initTRPC.create();

export const appRouter = t.router({
  user: userRouter,
});

// フロントエンドや shared パッケージへ型のみを export
export type AppRouter = typeof appRouter;
```

---

### Step 3: フロントエンド Client の利用

フロントエンド側で型安全なクライアントを初期化して利用します。

```typescript
// client/src/lib/trpc.ts
import { createTRPCReact } from '@trpc/react-query';
import type { AppRouter } from '@scope/backend-types';

export const trpc = createTRPCReact<AppRouter>();
```

```tsx
// client/src/components/UserProfile.tsx
import React from 'react';
import { trpc } from '../lib/trpc';

export function UserProfile({ userId }: { userId: string }) {
  const { data, isLoading } = trpc.user.getUserById.useQuery({ id: userId });

  if (isLoading) return <div>Loading...</div>;
  return <div>{data?.name}</div>;
}
```

---

### Step 4: 動的探索による検証プロシージャ

バックエンドおよびフロントエンドの各 `package.json`（`scripts`）を動的に確認し、型チェックとビルドを実行して型不整合がないか検証します。

```bash
# 対象パッケージの scripts を確認
cat apps/<server-or-client>/package.json | grep -A 10 '"scripts"'

# 型整合性の確認
pnpm --filter <server-package> run typecheck
pnpm --filter <client-package> run typecheck

# ビルド検証
pnpm --filter <server-package> run build
pnpm --filter <client-package> run build
```
