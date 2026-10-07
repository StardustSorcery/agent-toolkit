# NestJS Architecture & Monorepo Guidelines

本ドキュメントは、NestJS を採用した大規模バックエンドサービスにおけるアーキテクチャ設計・実装指針を定めた正本規約です。
NestJS monorepo mode（`nest-cli.json`）を基盤とし、6 レイヤ構造、Zod によるドメインモデリング、Repository と DAO の責務分離、Ports & Adapters（ヘキサゴナルアーキテクチャ）パターン、および厳格な DI・命名規則を定義します。

---

## 1. モノレポ構造と基本方針

### 1.1 NestJS Monorepo Mode
バックエンドサービス群は、モノレポ（pnpm workspace 等）配下の `backend-services/` パッケージに集約され、`nest-cli.json` に基づく **NestJS monorepo mode** で管理されます。
コードベースは `apps/`（実行可能アプリケーション）と `libs/`（再利用可能なライブラリ・レイヤ）に明確に分割されます。

```text
backend-services/
├── nest-cli.json               # NestJS monorepo 定義（applications と libraries の管理）
├── tsconfig.json               # パスエイリアス（@app/*）定義
├── package.json
├── apps/
│   ├── api/                    # REST / GraphQL API サーバー
│   ├── bff/                    # Backend for Frontend (tRPC / REST)
│   ├── realtime/               # WebSocket / SSE サーバー
│   └── batch/                  # CLI コマンド、Cron ジョブ、ワーカー
└── libs/
    ├── domain/                 # コアビジネスロジック、エンティティ、Repository IF
    ├── usecase-*/              # ユースケースサービス、Guard、Adaptor IF
    ├── infra-core/             # Repository 実装、DAO IF
    ├── infra-*/                # 技術固有のインフラ具象実装（DAO 実装、Adaptor 実装）
    ├── lib-*/                  # 接続ライフサイクル管理（NestJS 動的モジュールラッパー）
    └── lib-test-*/             # 単体・結合テスト用モック / インメモリユーティリティ
```

### 1.2 パスエイリアス解決 (`tsconfig.json`)
各ライブラリ間の参照は、`tsconfig.json` の `paths` エイリアス（`@app/<lib-name>`）を介して行います。相対パスによる深い階層の import（例: `../../libs/...`）は禁止します。

```json
{
  "compilerOptions": {
    "baseUrl": "./",
    "paths": {
      "@app/domain": ["libs/domain/src"],
      "@app/domain/*": ["libs/domain/src/*"],
      "@app/usecase-account": ["libs/usecase-account/src"],
      "@app/usecase-account/*": ["libs/usecase-account/src/*"],
      "@app/infra-core": ["libs/infra-core/src"],
      "@app/infra-core/*": ["libs/infra-core/src/*"],
      "@app/infra-postgresql": ["libs/infra-postgresql/src"],
      "@app/infra-postgresql/*": ["libs/infra-postgresql/src/*"]
    }
  }
}
```

---

## 2. 6 レイヤ構造と責務境界

システムは関心の分離と依存性逆転の原則 (DIP) に基づき、以下の 6 つのレイヤ（+ テストユーティリティ層）に分割されます。

| レイヤ名 | ディレクトリ | 主な構成要素 | 責務と特徴 |
| :--- | :--- | :--- | :--- |
| **Presentation & Composition Root** | `apps/*` | `main.ts`, `AppModule`, Controllers, DTOs, tRPC Routers, CLI Commands | アプリケーションのエントリーポイント。HTTP/RPC/CLI リクエストの受付、バリデーション、レスポンス整形。**Composition Root** として各モジュールを import し、DI コンテナを構築する。 |
| **Domain** | `libs/domain` (または `domain-*`) | Entities, Value Objects, Enums, Zod Schemas, Domain Repository Interfaces | 最もコアなビジネスロジック・不変条件の定義。**外部レイヤやフレームワーク（NestJS デコレータ等）に一切依存しない**純粋な TypeScript。永続化ストレージと親和性の高い Zod Schema + 純粋関数でモデリングする。 |
| **Application (Usecase)** | `libs/usecase-*` | Usecase Services, Guards, Interceptors, Adaptor Interfaces (Ports) | 機能・ドメイン境界単位（例: `usecase-account`, `usecase-auth`）で分割。ドメインモデルを協調させた業務フローのオーケストレーション、認可、並行性制御。外部サービス呼び出しは自身の Adaptor Interface（Ports）を介する。 |
| **Infra Core** | `libs/infra-core` | Repository Implementations, DAO Interfaces | インフラ共通層。**Domain Repository Interface と DAO を紐付ける中継レイヤ**。Domain Repository の具象実装を持ち、1 つ以上の DAO を呼び出してドメインエンティティを生成・永続化する。 |
| **Infra Tech** | `libs/infra-*` | DAO Implementations, Adaptor Implementations, Converters/Transformers | 技術固有のインフラ具象実装（`infra-postgresql`, `infra-redis`, `infra-mongodb`, `infra-s3` 等）。DAO Interface や Adaptor Interface を具象化し、DB・外部 API とのデータ変換（Converter）および実際の CRUD / 通信を担当。 |
| **Low-Level Drivers** | `libs/lib-*` | Dynamic Modules (`forRootAsync`), Connection Wrappers | インフラ・DB 接続のライフサイクル管理（`lib-postgresql`, `lib-redis` 等）。NestJS の動的モジュールを備えた raw クライアントの接続ラッパー。 |
| **Test Utilities** | `libs/lib-test-*` | Mock Clients, In-Memory Connections, Test Helpers | 単体テスト・結合テスト用のモックやインメモリ実装。各レイヤのテストコード（`*.spec.ts`, `*.e2e-spec.ts`）からのみ参照し、**本番コードへの import は厳禁**。 |

---

## 3. レイヤ依存ルール (Dependency Rules)

### 3.1 依存・実装関係の図式化

```mermaid
graph TD
    subgraph Presentation["Presentation & Composition Root Layer"]
        Apps["apps/*<br/>(Controllers / DTOs / AppModule / CLI)"]
    end

    subgraph Application["Application Layer"]
        Usecase["libs/usecase-*<br/>(Services / Guards / Adaptor IF)"]
    end

    subgraph CoreDomain["Domain Layer"]
        Domain["libs/domain<br/>(Entities / Zod Schemas / Repository IF)"]
    end

    subgraph Infrastructure["Infrastructure Layer"]
        InfraCore["libs/infra-core<br/>(Repository Impl / DAO IF)"]
        InfraTech["libs/infra-*<br/>(DAO Impl / Adaptor Impl / Converters)"]
        LibDrivers["libs/lib-*<br/>(Dynamic Modules / Driver Wrappers)"]
    end

    subgraph TestLayer["Test Utilities Layer (Non-Production)"]
        LibTest["libs/lib-test-*<br/>(Mock Clients / In-Memory Connections)"]
    end

    %% すべてのレイヤから Domain への依存
    Apps -->|依存| Domain
    Usecase -->|依存| Domain
    InfraCore -->|依存| Domain
    InfraTech -->|依存| Domain
    LibDrivers -->|依存| Domain
    LibTest -->|依存| Domain

    %% レイヤ間依存関係 (Dependency)
    Apps -->|依存| Usecase
    Apps -.->|Composition Root として import| InfraCore
    Apps -.->|Composition Root として import| InfraTech
    InfraCore -->|依存 (DAO IF利用)| InfraCore
    InfraTech -->|依存 (DAO IF参照)| InfraCore
    InfraTech -->|依存 (Driver利用)| LibDrivers
    LibTest -->|依存| LibDrivers

    %% 実装関係 (Realization / Implementation)
    InfraCore ==>|実装 (Repository IF)| Domain
    InfraTech ==>|実装 (DAO IF)| InfraCore
    InfraTech ==>|実装 (Adaptor IF)| Usecase

    %% テスト時のみの依存 (Test Dependency)
    Apps -.->|テスト時依存| LibTest
    Usecase -.->|テスト時依存| LibTest
    InfraTech -.->|テスト時依存| LibTest
```

### 3.2 厳格な遵守ルール
1. **Domain の完全保護 (Framework-Agnostic)**:
   - `libs/domain` は最内周に位置し、外部のいかなるレイヤ・ライブラリにも依存してはなりません。
   - `@Injectable()`, `@Inject()` などの NestJS デコレータや `@nestjs/*` パッケージの import は一切禁止します。
2. **依存性逆転の徹底 (DIP)**:
   - `libs/usecase-*` は `libs/infra-*` や `libs/infra-core` に直接依存してはなりません。外部への要求は自層で定義した **Adaptor Interface（Ports）** を介して行い、具象実装は `libs/infra-*` が行います。
3. **Repository と DAO の階層分離**:
   - `libs/infra-core` は Domain Repository Interface を実装し、DAO Interface を呼び出します。
   - `libs/infra-*` は DAO Interface を実装し、実際のデータベース操作を行います。
4. **Composition Root の責務**:
   - `apps/*` の `AppModule` は Composition Root として、各 `usecase-*` モジュールや `infra-*` モジュールを import して依存関係を解決します。
   - コントローラーなどのプレゼンテーションコード自体は `usecase-*` および `domain` にのみ直接依存し、インフラ具象クラスに直接依存してはなりません。
5. **テストコードの境界隔離**:
   - `libs/lib-test-*` はテストコード（`*.spec.ts`, `*.e2e-spec.ts`）からのみ参照し、本番コード（`main.ts` や各レイヤのプロダクションモジュール）からの import は厳禁です。

---

## 4. ドメインモデリング規約 (Zod Schema & 純粋 Helper 関数)

ドメインモデルは、永続化ストレージ（RDB / NoSQL）との直接入出力・シリアライズを容易にし、かつ実行時バリデーションと型安全性を両立するため、**Zod Schema を軸にしたデータ構造（型）と純粋 Helper 関数** の組み合わせで定義します。

- **Entity はメソッドを持たない（Pure Data Structure）**: クラスインスタンスのメソッドではなく、プレーンな JavaScript オブジェクトとして表現します。
- **操作は純粋関数（Helper）として提供**: Entity に対する状態判定や計算ロジックは、同名の const オブジェクト内の純粋関数として公開します。

### 実装例 (`libs/domain/src/account/entities/account.entity.ts`)

```typescript
import { z } from 'zod';

const schema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  name: z.string().min(1),
  role: z.enum(['ADMIN', 'USER', 'GUEST']),
  isSuspended: z.boolean(),
  suspendedAt: z.date().nullable(),
  createdAt: z.date(),
  updatedAt: z.date(),
});

// ドメインエンティティの型定義
export type Account = z.infer<typeof schema>;

// スキーマと純粋 helper 関数を同一識別子で export
export const Account = {
  schema,
  isSuspended: (account: Pick<Account, 'isSuspended' | 'suspendedAt'>): boolean => {
    return account.isSuspended || account.suspendedAt !== null;
  },
  canPerformAdminAction: (account: Pick<Account, 'role' | 'isSuspended' | 'suspendedAt'>): boolean => {
    return account.role === 'ADMIN' && !Account.isSuspended(account);
  },
} as const;
```

---

## 5. Repository と DAO の責務分離アーキテクチャ

データの永続化・取得において、**ドメインの関心（Repository）** と **データアクセスの関心（DAO）** を完全に分離します。

```text
libs/domain/src/*/repositories/
└── account.repository.ts          # [Domain Repository IF] ドメインの関心（ビジネス単位の取得・永続化）
         ▲
         │ implements
libs/infra-core/src/repositories/
└── account.repository.impl.ts     # [Repository Impl] 1つ以上の DAO を協調させ、Domain Entity を再構築
         │
         │ uses (Injects)
libs/infra-core/src/daos/
└── account.dao.ts                 # [DAO IF] ストレージの関心（テーブル/コレクション単位の CRUD）
         ▲
         │ implements
libs/infra-postgresql/src/daos/
└── account.dao.impl.ts            # [DAO Impl] ORM / クエリビルダ / Raw SQL による具象操作
```

### 5.1 各要素の責務定義

1. **Domain Repository Interface (`libs/domain`)**:
   - ビジネス視点でのドメインエンティティの集合操作・永続化インターフェース（抽象クラス）。
   - 特定のデータベース構造やクエリ技術には関知しない。
2. **DAO Interface (`libs/infra-core`)**:
   - 永続化ストレージ固有のテーブルやコレクションに対応する CRUD インターフェース（抽象クラス）。
3. **Domain Repository Implementation (`libs/infra-core`)**:
   - Domain Repository Interface の具象実装。
   - 1 つまたは複数の DAO を呼び出し、取得したデータをドメインエンティティに再構築（必要に応じて Converter を利用）して提供する。
4. **DAO Implementation (`libs/infra-*`)**:
   - DAO Interface の具象実装。
   - PostgreSQL, MongoDB 等のドライバや ORM を用いてデータベースアクセスを実行する。
5. **Converter / Transformer (`libs/infra-*`)**:
   - データベースレコード（Entity / Document）とドメインモデル（Domain Entity）間の双方向変換を行う純粋関数群。

---

## 6. Ports & Adapters (Adaptor Interface) 規約

ユースケースが外部サービス（認証プロバイダ、メール送信、外部 API 等）を呼び出す場合、ユースケース層で **Adaptor Interface（Ports）** を定義し、インフラ層で具象アダプターを実装します。

### 6.1 Adaptor Interface 定義 (`libs/usecase-account/src/adaptors/notification.adapter.ts`)

```typescript
export abstract class NotificationAdapter {
  abstract sendEmail(to: string, subject: string, body: string): Promise<void>;
  abstract sendPushNotification(userId: string, message: string): Promise<void>;
}
```

### 6.2 Usecase Service での利用 (`libs/usecase-account/src/services/account.service.ts`)

```typescript
import { Injectable } from '@nestjs/common';
import { AccountRepository } from '@app/domain/account/repositories/account.repository';
import { NotificationAdapter } from '../adaptors/notification.adapter';
import { Account } from '@app/domain/account/entities/account.entity';

@Injectable()
export class AccountService {
  constructor(
    private readonly accountRepository: AccountRepository,
    private readonly notificationAdapter: NotificationAdapter,
  ) {}

  async suspendAccount(accountId: string, reason: string): Promise<void> {
    const account = await this.accountRepository.findById(accountId);
    if (!account) {
      throw new Error('Account not found');
    }

    await this.accountRepository.updateSuspensionStatus(accountId, true);
    await this.notificationAdapter.sendEmail(account.email, 'Account Suspended', reason);
  }
}
```

---

## 7. NestJS DI (Dependency Injection) パターンと命名規約

### 7.1 抽象クラスによるポート定義
TypeScript の `interface` はトランスパイル時に型情報が消去されるため、実行時の DI トークンとして機能しません。
そのため、すべてのポート（Domain Repository IF, DAO IF, Adaptor IF）は **抽象クラス (`abstract class`)** として定義します。これにより、型注釈と DI トークンを同一シンボルで扱えます。

### 7.2 具象クラスおよびファイルサフィックス (`*.impl.ts` / `*Impl`)
インターフェースまたは抽象クラスを実装する具象クラスは、**必ずクラス名に `Impl`、ファイル名に `.impl.ts`** を付与します（テストコードは `.impl.spec.ts`）。

| 種別 | 抽象クラス定義場所 | 具象クラス名 | 具象ファイル名 | 配置場所 |
| :--- | :--- | :--- | :--- | :--- |
| **Domain Repository** | `AccountRepository` (`libs/domain`) | `AccountRepositoryImpl` | `account.repository.impl.ts` | `libs/infra-core` |
| **DAO** | `AccountDao` (`libs/infra-core`) | `AccountDaoImpl` | `account.dao.impl.ts` | `libs/infra-*` |
| **Adaptor** | `NotificationAdapter` (`libs/usecase-*`) | `SmtpNotificationAdapterImpl` | `smtp-notification.adapter.impl.ts` | `libs/infra-*` |
| **Adaptor** | `AuthTokenAdapter` (`libs/usecase-*`) | `JwtAuthTokenAdapterImpl` | `jwt-auth-token.adapter.impl.ts` | `libs/infra-*` |

### 7.3 モジュールバインディングパターン

#### パターン A: 抽象クラスを直接トークンとするバインド（推奨）

```typescript
// libs/infra-postgresql/src/module.ts
import { Module } from '@nestjs/common';
import { AccountDao } from '@app/infra-core/daos/account.dao';
import { AccountDaoImpl } from './daos/account.dao.impl';

@Module({
  providers: [
    AccountDaoImpl,
    {
      provide: AccountDao,
      useClass: AccountDaoImpl,
    },
  ],
  exports: [AccountDao],
})
export class InfraPostgresqlModule {}
```

#### パターン B: `useExisting` によるシングルトン共有

```typescript
// libs/infra-core/src/module.ts
import { Module } from '@nestjs/common';
import { AccountRepository } from '@app/domain/account/repositories/account.repository';
import { AccountRepositoryImpl } from './repositories/account.repository.impl';

@Module({
  providers: [
    AccountRepositoryImpl,
    {
      provide: AccountRepository,
      useExisting: AccountRepositoryImpl,
    },
  ],
  exports: [AccountRepository],
})
export class InfraCoreModule {}
```

### 7.4 インジェクション側の記述

```typescript
@Injectable()
export class AccountRepositoryImpl implements AccountRepository {
  constructor(
    @Inject(AccountDao)
    private readonly accountDao: AccountDao,
  ) {}

  async findById(id: string): Promise<Account | null> {
    return this.accountDao.findById(id);
  }
}
```

### 7.5 その他のファイル命名規約
- **Entity**: `*.entity.ts` (例: `account.entity.ts`)
- **Value Object**: `*.value.ts` (例: `email.value.ts`)
- **Command**: `*.command.ts` (例: `create-account.command.ts`)
- **Request / Response DTO**: `*.request.dto.ts` / `*.response.dto.ts`
- **Guard / Interceptor**: `*.guard.ts` / `*.interceptor.ts`
- **Usecase Service**: `*.service.ts` (例: `account.service.ts`)
- **Controller**: `*.controller.ts` (例: `account.controller.ts`)
- **Converter**: `*.converter.ts` (例: `account.converter.ts`)

---

## 8. テスト規約とモック方針

1. **単体テスト (`*.spec.ts`)**:
   - `Test.createTestingModule` を使用し、依存する抽象クラス（Repository, DAO, Adapter）は Jest モックオブジェクト（`jest.Mocked<AccountDao>` 等）やテストライブラリ（`libs/lib-test-*`）で差し替えます。
   - 外部インフラ（DB や外部 API）を起動せず、ミリ秒単位で高速かつ決定論的に実行可能であることを担保します。
2. **結合 / E2E テスト (`*.e2e-spec.ts`)**:
   - `apps/*/test/` 配下に配置します。
   - テスト環境で立ち上げた実際のコンテナ（PostgreSQL, Redis 等）と接続し、HTTP / RPC リクエストから DB 永続化までのエンドツーエンドシナリオを検証します。
3. **Test Utilities (`libs/lib-test-*`) の厳格な運用**:
   - 単体・結合テスト用のインメモリ実装やテストヘルパーを提供します。
   - テストコード（`*.spec.ts`, `*.e2e-spec.ts`）からのみ参照し、本番モジュールへの混入を CI の lint / ビルドチェックで防止します。
