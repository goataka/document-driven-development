# ADR-007: DynamoDB Toolboxの導入

## ステータス

提案中（Proposed）

## 背景

DynamoDBとのやり取りをより型安全かつ効率的に行うため、DynamoDB Toolboxの導入を検討しています。

### DynamoDB Toolbox とは

AWS SDK for JavaScriptをラップし、TypeScriptの型安全性を提供するライブラリです。エンティティ定義、バリデーション、クエリビルダーなどの機能を提供します。

### 現在の実装（AWS SDK直接使用）

```typescript
const result = await docClient.send(
  new GetCommand({
    TableName: 'users',
    Key: { userId: '123' },
  })
);
const user = result.Item as User; // 型安全性なし
```

### DynamoDB Toolbox を使用した場合

```typescript
const user = await User.get({ userId: '123' });
// 完全な型安全性
```

## 決定事項

**検討中**: DynamoDB Toolboxの導入を検討するが、プロジェクトの規模と複雑さに応じて判断する。

### 導入を推奨するケース

1. **型安全性重視**: TypeScriptの型チェックを最大限活用したい
2. **複雑なデータモデル**: 多数のエンティティと関係性がある
3. **Single Table Design**: 複雑なアクセスパターンを管理したい
4. **バリデーション**: スキーマバリデーションが必要

### AWS SDK直接使用を推奨するケース

1. **シンプルなデータモデル**: エンティティ数が少ない（2-3個）
2. **学習コスト**: チームがAWS SDKに習熟している
3. **柔軟性**: 低レベルAPIの直接制御が必要
4. **依存関係最小化**: 外部ライブラリを減らしたい

## 結果

### メリット（DynamoDB Toolbox 導入）

- TypeScriptの型安全性向上
- スキーマ定義による明確なデータモデル
- バリデーション機能
- クエリビルダーの使いやすさ
- Single Table Design のサポート

### デメリット

- 学習コストの増加
- 追加の依存関係
- AWS SDK の更新に追従する必要
- コミュニティサイズが小さい

### パフォーマンス影響

- 最小限のオーバーヘッド
- ランタイムパフォーマンスへの影響は軽微

## 実装例

### AWS SDK直接使用（現在）

```typescript
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, GetCommand, PutCommand } from '@aws-sdk/lib-dynamodb';

const client = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client);

// 取得
const result = await docClient.send(
  new GetCommand({
    TableName: 'users',
    Key: { userId: '123' },
  })
);
const user = result.Item as User; // 型安全性が低い

// 保存
await docClient.send(
  new PutCommand({
    TableName: 'users',
    Item: {
      userId: '123',
      email: 'user@example.com',
      name: 'John Doe',
    },
  })
);
```

### DynamoDB Toolbox を使用（検討案）

```typescript
import { Entity, Table } from 'dynamodb-toolbox';
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient } from '@aws-sdk/lib-dynamodb';

// テーブル定義
const MyTable = new Table({
  name: 'users',
  partitionKey: 'PK',
  sortKey: 'SK',
  DocumentClient: DynamoDBDocumentClient.from(new DynamoDBClient({})),
});

// エンティティ定義
const User = new Entity({
  name: 'User',
  attributes: {
    userId: { partitionKey: true },
    email: { type: 'string', required: true },
    name: { type: 'string', required: true },
    companyCode: { type: 'string' },
    createdAt: { type: 'string', default: () => new Date().toISOString() },
  },
  table: MyTable,
} as const);

// 型安全な操作
const user = await User.get({ userId: '123' }); // 完全な型推論
await User.put({
  userId: '123',
  email: 'user@example.com',
  name: 'John Doe',
}); // 型チェック
```

### バリデーション例

```typescript
// DynamoDB Toolbox は自動的にバリデーション
try {
  await User.put({
    userId: '123',
    email: 'invalid-email', // バリデーションエラー
    // name: 必須フィールドが不足 -> エラー
  });
} catch (error) {
  console.error('Validation failed:', error);
}
```

## 代替案

### 1. 自前のラッパー実装

```typescript
class UserRepository {
  private readonly tableName = 'users';

  async get(userId: string): Promise<User | null> {
    const result = await docClient.send(
      new GetCommand({
        TableName: this.tableName,
        Key: { userId },
      })
    );
    return result.Item ? this.mapToUser(result.Item) : null;
  }

  private mapToUser(item: any): User {
    return {
      userId: item.userId,
      email: item.email,
      name: item.name,
      companyCode: item.companyCode,
    };
  }
}
```

### 2. TypeORM DynamoDB Adapter（実験的）

TypeORMのカスタムドライバーを実装する案もありますが、公式サポートがないため推奨しません。

## 移行戦略

### Phase 1（現在）: AWS SDK直接使用
- シンプルな実装
- 学習コスト低い

### Phase 2（中期）: リポジトリパターン
- データアクセスを抽象化
- 型安全性を手動で確保

### Phase 3（将来）: DynamoDB Toolbox 導入
- エンティティが増加した場合
- Single Table Design 採用時

## コスト・パフォーマンス

- DynamoDB Toolbox: 無料のOSSライブラリ
- パフォーマンスオーバーヘッド: 最小限（< 1ms）
- 開発効率: 向上（型安全性、バリデーション）

## 学習リソース

- [DynamoDB Toolbox Documentation](https://www.dynamodbtoolbox.com/)
- [GitHub Repository](https://github.com/jeremydaly/dynamodb-toolbox)
- [AWS SDK for JavaScript v3](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/)

## 関連ドキュメント

- [データベース設計](../../docs/implement/DATABASE.md)
- [データベース管理方針](../../docs/deploy/DATABASE.md)
- [ADR-006: Single Table Design](./006-dynamodb-single-table-design.md)

## 更新履歴

- 2024-12-21: 初版作成
