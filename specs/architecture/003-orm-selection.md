# ADR-003: ORMの選定

## ステータス

提案中（Proposed）

## 背景

バックエンドアプリケーションにおいて、データベースとのやり取りを型安全かつ効率的に行うため、ORM（Object-Relational Mapping）ライブラリの採用を検討しています。

DynamoDBを主要データベースとして採用していますが、ローカル開発やCI環境でのテスト、将来的な拡張性を考慮し、以下の候補を検討しています：

- **TypeORM**: NestJSとの統合が良好なデコレータベースのORM
- **Prisma**: 型安全性重視のモダンなORM

## 決定事項

**推奨**: TypeORM を第一選択とし、型安全性を最優先する場合は Prisma も検討可能とする。

### TypeORM を推奨する理由

1. **NestJS統合**: NestJSとのシームレスな統合
2. **デコレータベース**: TypeScriptのデコレータを活用した直感的なAPI
3. **マイグレーション**: データベーススキーマのバージョン管理
4. **リレーション**: 複雑なリレーションシップの表現が可能
5. **実績**: 多くのプロジェクトで採用実績

### Prisma を検討すべきケース

1. **型安全性最優先**: Prisma Clientの自動生成による完全な型安全性
2. **モダンなAPI**: より直感的で読みやすいクエリAPI
3. **Prisma Studio**: GUIベースのデータベース管理ツール
4. **マイグレーション**: より洗練されたマイグレーションシステム

## 結果

### メリット（TypeORM 採用の場合）

- NestJSのエコシステムとの親和性
- デコレータによる宣言的なエンティティ定義
- Active Record / Data Mapper パターンの選択肢
- 豊富なドキュメントとコミュニティサポート

### デメリット

- Prismaと比較すると型安全性がやや劣る
- クエリビルダーがやや冗長になる場合がある
- パフォーマンスチューニングが必要な場合がある

### DynamoDB との統合

TypeORMは主にリレーショナルデータベース向けですが、以下のアプローチで対応します：

1. **AWS SDK 直接使用**: DynamoDBにはAWS SDKを直接使用
2. **TypeORMは補助的に使用**: ローカル開発・テスト環境でのみ使用
3. **データアクセスレイヤー抽象化**: リポジトリパターンで実装を隠蔽

## 実装例

### TypeORM の基本的な使用例

```typescript
// entity/user.entity.ts
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string;

  @Column()
  name: string;

  @Column({ nullable: true })
  companyCode?: string;
}

// user.repository.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';

@Injectable()
export class UserRepository {
  constructor(
    @InjectRepository(User)
    private readonly repository: Repository<User>,
  ) {}

  async findByEmail(email: string): Promise<User | null> {
    return this.repository.findOne({ where: { email } });
  }

  async create(data: Partial<User>): Promise<User> {
    const user = this.repository.create(data);
    return this.repository.save(user);
  }
}
```

### DynamoDB との統合パターン

```typescript
// DynamoDB用の抽象化
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, GetCommand } from '@aws-sdk/lib-dynamodb';

@Injectable()
export class DynamoDBUserRepository {
  private readonly docClient: DynamoDBDocumentClient;

  constructor() {
    const client = new DynamoDBClient({});
    this.docClient = DynamoDBDocumentClient.from(client);
  }

  async findByEmail(email: string): Promise<User | null> {
    const result = await this.docClient.send(
      new GetCommand({
        TableName: 'users',
        Key: { email },
      })
    );
    return result.Item as User;
  }
}
```

## 関連ドキュメント

- [システムアーキテクチャ設計書](../../docs/implement/README.md)
- [バックエンド設計](../../docs/implement/BACKEND.md)
- [データベース設計](../../docs/implement/DATABASE.md)

## 更新履歴

- 2024-12-21: 初版作成
