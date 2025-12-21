# ADR-008: DynamoDB DAX（Accelerator）の導入

## ステータス

提案中（Proposed）

## 背景

高頻度の読み取りアクセスが発生する場合、DynamoDB のレスポンスタイムをさらに改善するため、DynamoDB Accelerator (DAX) の導入を検討しています。

### DAX とは

DynamoDB用のフルマネージドインメモリキャッシュサービスで、マイクロ秒レベルのレスポンスタイムを実現します。

### パフォーマンス比較

- **DynamoDB直接**: 一桁ミリ秒（5-10ms）
- **DAX経由**: マイクロ秒レベル（< 1ms）

## 決定事項

**オプション**: 高頻度読み取りが必要な場合のみ、DAXの導入を検討する。初期実装では不要。

### DAX を導入すべきケース

1. **高頻度読み取り**: 秒間1000リクエスト以上の読み取り
2. **レイテンシ要件**: サブミリ秒のレスポンスが必要
3. **読み取り重視**: 読み取りが書き込みの10倍以上
4. **キャッシュ効率**: 同じデータへの繰り返しアクセスが多い
5. **コスト最適化**: DynamoDBの読み取りコストが高い

### DAX なしで運用する場合

1. **通常のDynamoDB**: 十分なパフォーマンス（5-10ms）
2. **アプリケーションレベルキャッシュ**: メモリ内キャッシュ
3. **Redis**: 柔軟なキャッシュ戦略

## 結果

### メリット（DAX 導入）

- マイクロ秒レベルのレスポンス（10倍以上の高速化）
- DynamoDB読み取りコストの削減
- フルマネージド（運用負荷なし）
- DynamoDBとの互換性（コード変更最小）
- 自動フェイルオーバー

### デメリット

- 追加コスト（月額$100-500以上）
- 書き込み整合性の考慮が必要
- キャッシュ管理の複雑さ
- データの鮮度に遅延が発生する可能性

### パフォーマンス効果

```
ベンチマーク例:
- DynamoDB: 10ms/request
- DAX: 0.5ms/request
- 改善率: 20倍高速化
```

### コスト分析

#### DAXあり
- DAX クラスタ (dax.t3.small × 2ノード): 月額 $230
- DynamoDB 読み取り削減: -$50
- **実質コスト増**: 月額 $180

#### DAXなし
- DynamoDB 読み取りコスト: 月額 $100
- **合計**: 月額 $100

**結論**: 高トラフィックでない限り、コスト対効果が低い

## 実装例

### DAX なし（現在）

```typescript
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, GetCommand } from '@aws-sdk/lib-dynamodb';

const client = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client);

const user = await docClient.send(
  new GetCommand({
    TableName: 'users',
    Key: { userId: '123' },
  })
);
```

### DAX 使用（検討案）

```typescript
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, GetCommand } from '@aws-sdk/lib-dynamodb';
import AmazonDaxClient from 'amazon-dax-client';

// DAXクライアントの作成
const daxClient = new AmazonDaxClient({
  endpoints: [process.env.DAX_ENDPOINT],
  region: 'ap-northeast-1',
});

const docClient = DynamoDBDocumentClient.from(daxClient);

// 通常のDynamoDBと同じコード
const user = await docClient.send(
  new GetCommand({
    TableName: 'users',
    Key: { userId: '123' },
  })
);
// キャッシュヒット時: < 1ms
// キャッシュミス時: 5-10ms（DynamoDBから取得）
```

### 環境別切り替え

```typescript
class DatabaseClientFactory {
  static create() {
    if (process.env.NODE_ENV === 'production' && process.env.DAX_ENDPOINT) {
      // 本番環境でDAXを使用
      return new AmazonDaxClient({
        endpoints: [process.env.DAX_ENDPOINT],
        region: process.env.AWS_REGION,
      });
    }
    // 開発・ステージング環境では通常のDynamoDB
    return new DynamoDBClient({ region: process.env.AWS_REGION });
  }
}
```

## 導入判断基準

### 導入を推奨する指標

| 指標 | 閾値 | 現状 | 判定 |
|------|------|------|------|
| 秒間読み取りリクエスト | > 1000 | < 100 | ❌ 不要 |
| 平均レスポンスタイム | > 10ms | 5-8ms | ✅ OK |
| 月間読み取りコスト | > $200 | < $50 | ❌ 不要 |
| キャッシュヒット率 | > 70% | N/A | N/A |

**結論**: 現状では DAX 導入は不要

## 段階的導入戦略

### Phase 1（現在）: DynamoDB直接アクセス
- コスト: 低い
- パフォーマンス: 十分
- 運用: シンプル

### Phase 2（高トラフィック時）: Redis キャッシュ
- コスト: 中程度
- パフォーマンス: 改善
- 柔軟性: 高い

### Phase 3（超高トラフィック時）: DAX 導入
- コスト: 高い
- パフォーマンス: 最高
- 運用: マネージド

## 代替案

### 1. Redis キャッシュ

```typescript
class CachedUserRepository {
  constructor(
    private readonly redis: Redis,
    private readonly dynamodb: DynamoDBDocumentClient,
  ) {}

  async get(userId: string): Promise<User | null> {
    // Redisキャッシュを確認
    const cached = await this.redis.get(`user:${userId}`);
    if (cached) return JSON.parse(cached);

    // DynamoDBから取得
    const result = await this.dynamodb.send(
      new GetCommand({
        TableName: 'users',
        Key: { userId },
      })
    );

    // Redisにキャッシュ
    if (result.Item) {
      await this.redis.setex(
        `user:${userId}`,
        3600, // 1時間
        JSON.stringify(result.Item)
      );
    }

    return result.Item as User;
  }
}
```

**メリット**: 柔軟性、低コスト  
**デメリット**: 管理の複雑さ

### 2. アプリケーションレベルキャッシュ

```typescript
import NodeCache from 'node-cache';

const cache = new NodeCache({ stdTTL: 600 }); // 10分

async function getCachedUser(userId: string): Promise<User | null> {
  const cached = cache.get<User>(userId);
  if (cached) return cached;

  const user = await getUserFromDynamoDB(userId);
  if (user) cache.set(userId, user);
  return user;
}
```

**メリット**: 最もシンプル、コストゼロ  
**デメリット**: 単一インスタンスのみ、スケーラビリティ制限

## 監視とアラート

DAX導入時の監視項目：

- **キャッシュヒット率**: 目標 > 80%
- **レイテンシ**: 目標 < 1ms
- **スロットリング**: 0件
- **クラスタヘルス**: 健全

## 学習リソース

- [Amazon DynamoDB Accelerator (DAX)](https://aws.amazon.com/dynamodb/dax/)
- [DAX Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DAX.html)
- [DAX Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DAX.best-practices.html)

## 関連ドキュメント

- [データベース管理方針](../../docs/deploy/DATABASE.md)
- [ADR-004: Redis キャッシュの導入](./004-redis-cache-usage.md)

## 更新履歴

- 2024-12-21: 初版作成
