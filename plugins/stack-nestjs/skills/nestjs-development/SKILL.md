---
name: nestjs-development
description: >-
  Use this skill when implementing use cases, domain entities, ports, or repository adapters
  in a NestJS application following Clean Architecture.
---

# NestJS Clean Architecture Development Skill

本スキルは、NestJS バックエンドにおいて、ドメインエンティティ、ポート定義、具象アダプタ（`*Impl`）、ユースケースサービスを追加・実装し、検証するための手順書です。

---

## 開発プロシージャ

### Step 1: ドメインエンティティおよびポート抽象クラスの定義

外部フレームワークに依存しない純粋なドメインエンティティと、ユースケース層から利用するポート抽象クラスを作成します。

```typescript
// 1. domain/src/entities/item.entity.ts
export class Item {
  constructor(
    readonly id: string,
    readonly name: string,
    readonly createdAt: Date,
  ) {}
}

// 2. ports/src/item.repository.ts
import { Item } from './entities/item.entity';

export abstract class ItemRepository {
  abstract findById(id: string): Promise<Item | null>;
  abstract save(item: Item): Promise<void>;
}
```

---

### Step 2: 具象実装クラス (`*Impl`) の作成

インフラ層にてポート抽象クラスを継承した具象クラスを実装します。ファイル名には必ず `.impl.ts`、クラス名には `Impl` を付与します。

```typescript
// infra/src/repositories/item.repository.impl.ts
import { Injectable } from '@nestjs/common';
import { ItemRepository } from '@scope/ports';
import { Item } from '@scope/domain';

@Injectable()
export class ItemRepositoryImpl extends ItemRepository {
  async findById(id: string): Promise<Item | null> {
    // データストアからの取得・ドメインエンティティへの変換
    return null;
  }

  async save(item: Item): Promise<void> {
    // 永続化処理
  }
}
```

---

### Step 3: NestJS モジュールへのプロバイダ登録

抽象クラスをトークンとして具象クラスをバインドします。

```typescript
// infra/src/item-infra.module.ts
import { Module } from '@nestjs/common';
import { ItemRepository } from '@scope/ports';
import { ItemRepositoryImpl } from './repositories/item.repository.impl';

@Module({
  providers: [
    ItemRepositoryImpl,
    {
      provide: ItemRepository,
      useExisting: ItemRepositoryImpl,
    },
  ],
  exports: [ItemRepository],
})
export class ItemInfraModule {}
```

---

### Step 4: 動的探索による検証プロシージャ

特定スクリプトの固定呼び出しを行わず、対象パッケージの `package.json`（`scripts`）を動的に確認してビルドおよびテストを実行します。

```bash
# 対象パッケージの scripts を確認
cat <package-dir>/package.json | grep -A 10 '"scripts"'

# 型検査およびビルド
pnpm --filter <package-name> run build

# 単体テストの実行
pnpm --filter <package-name> run test
```
