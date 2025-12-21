# ADR-010: DynamoDB グローバルテーブルの構成

## ステータス

提案中（Proposed）

## 背景

多地域展開や災害復旧（DR）戦略として、DynamoDB グローバルテーブルの導入を検討しています。

### グローバルテーブルとは

複数のAWSリージョン間でDynamoDBテーブルを自動的にレプリケーションし、低レイテンシのグローバルアクセスと高可用性を実現する機能です。

### ユースケース

1. **グローバルアプリケーション**: 複数地域のユーザーに低レイテンシでサービス提供
2. **災害復旧（DR）**: リージョン障害時の自動フェイルオーバー
3. **ローカルリード**: 各リージョンからの読み取りを高速化
4. **データ主権**: 地域ごとのデータ保存要件への対応

## 決定事項

**検討可能**: 多地域展開時にのみグローバルテーブルを検討する。初期実装では単一リージョン構成とする。

### グローバルテーブルを導入すべきケース

1. **多地域展開**: 複数の国・地域でサービスを提供
2. **低レイテンシ要件**: 各地域で10ms以下のレスポンスが必要
3. **高可用性**: 99.999%（ファイブナイン）の可用性が必要
4. **災害復旧**: リージョン障害時の自動フェイルオーバー
5. **コンプライアンス**: データローカライゼーション要件

### 単一リージョンで十分なケース

1. **国内限定サービス**: 日本国内のみでサービス提供
2. **コスト重視**: グローバルテーブルのコストが高い
3. **シンプル運用**: レプリケーション管理の複雑さを避けたい
4. **低トラフィック**: リージョン障害時のダウンタイムを許容できる

## 結果

### メリット（グローバルテーブル）

- **低レイテンシ**: 各リージョンでローカル読み取り（< 10ms）
- **高可用性**: マルチリージョン自動レプリケーション
- **自動フェイルオーバー**: リージョン障害時の自動切り替え
- **書き込みの自動レプリケーション**: 秒単位で同期
- **強い整合性**: 同一リージョン内では強い整合性

### デメリット

- **高コスト**: レプリケーションコストが2-3倍
- **結果整合性**: リージョン間は結果整合性（通常1秒以内）
- **競合解決**: 同時書き込みの競合管理が必要
- **運用複雑性**: マルチリージョン構成の管理

### コスト分析

#### 単一リージョン（東京）
- 書き込み: 100万リクエスト/月 → $1.25
- 読み取り: 1000万リクエスト/月 → $2.50
- ストレージ: 1GB → $0.25
- **合計**: 月額 $4

#### グローバルテーブル（東京 + シンガポール）
- 書き込み: 100万リクエスト/月 × 2 → $2.50
- レプリケーション: 100万書き込み → $1.88
- 読み取り: 1000万リクエスト/月 × 2 → $5.00
- ストレージ: 1GB × 2 → $0.50
- **合計**: 月額 $9.88（約2.5倍）

## 実装例

### グローバルテーブルの作成

```typescript
import { DynamoDBClient, CreateGlobalTableCommand } from '@aws-sdk/client-dynamodb';

const client = new DynamoDBClient({ region: 'ap-northeast-1' });

// グローバルテーブルの作成
const command = new CreateGlobalTableCommand({
  GlobalTableName: 'users',
  ReplicationGroup: [
    { RegionName: 'ap-northeast-1' }, // 東京
    { RegionName: 'ap-southeast-1' }, // シンガポール
    { RegionName: 'us-east-1' },      // バージニア北部
  ],
});

await client.send(command);
```

### リージョン別アクセス

```typescript
class GlobalDynamoDBService {
  private clients: Map<string, DynamoDBDocumentClient> = new Map();

  constructor() {
    // 各リージョンのクライアントを初期化
    const regions = ['ap-northeast-1', 'ap-southeast-1', 'us-east-1'];
    regions.forEach(region => {
      const client = new DynamoDBClient({ region });
      this.clients.set(region, DynamoDBDocumentClient.from(client));
    });
  }

  // ユーザーの地理的位置に基づいて最適なリージョンを選択
  getClientForUser(userLocation: string): DynamoDBDocumentClient {
    if (userLocation.startsWith('JP')) return this.clients.get('ap-northeast-1');
    if (userLocation.startsWith('SG')) return this.clients.get('ap-southeast-1');
    return this.clients.get('us-east-1'); // デフォルト
  }

  async getUser(userId: string, userLocation: string): Promise<User> {
    const client = this.getClientForUser(userLocation);
    const result = await client.send(
      new GetCommand({
        TableName: 'users',
        Key: { userId },
      })
    );
    return result.Item as User;
  }
}
```

### 競合解決戦略

```typescript
// Last Writer Wins (LWW) 戦略
interface UserWithTimestamp {
  userId: string;
  name: string;
  email: string;
  lastModified: number; // タイムスタンプ
  version: number;      // バージョン番号
}

async function updateUserWithConflictResolution(
  userId: string,
  updates: Partial<User>
): Promise<void> {
  const timestamp = Date.now();
  
  await docClient.send(
    new UpdateCommand({
      TableName: 'users',
      Key: { userId },
      UpdateExpression: 'SET #name = :name, lastModified = :timestamp, version = version + :inc',
      ConditionExpression: 'lastModified < :timestamp', // 新しいタイムスタンプのみ適用
      ExpressionAttributeNames: {
        '#name': 'name',
      },
      ExpressionAttributeValues: {
        ':name': updates.name,
        ':timestamp': timestamp,
        ':inc': 1,
      },
    })
  );
}
```

## 段階的導入戦略

### Phase 1（現在）: 単一リージョン（東京）
- コスト: 低
- レイテンシ: 日本国内で最適
- 運用: シンプル

### Phase 2（国内拡大）: 単一リージョン継続
- マルチAZ構成で高可用性
- バックアップ強化

### Phase 3（アジア展開）: グローバルテーブル導入
- 東京 + シンガポール
- アジア地域での低レイテンシ

### Phase 4（グローバル展開）: マルチリージョン
- 東京 + シンガポール + 米国
- 世界中で低レイテンシ

## 代替案

### 1. マルチリージョンアプリケーション（手動レプリケーション）

```typescript
// 書き込み時に手動で複数リージョンに書き込む
async function writeToMultipleRegions(user: User): Promise<void> {
  const regions = ['ap-northeast-1', 'ap-southeast-1'];
  
  await Promise.all(
    regions.map(region => {
      const client = getClientForRegion(region);
      return client.send(
        new PutCommand({
          TableName: 'users',
          Item: user,
        })
      );
    })
  );
}
```

**デメリット**: 複雑な実装、整合性管理が困難

### 2. リードレプリカ戦略

プライマリリージョンへの書き込みのみ、読み取りは各リージョンのキャッシュ（Redis）から行う。

**メリット**: コスト削減、シンプルな実装  
**デメリット**: キャッシュ整合性の管理

## 監視とアラート

グローバルテーブル運用時の監視項目：

- **レプリケーション遅延**: 目標 < 1秒
- **レプリケーション失敗**: 0件
- **リージョン別レイテンシ**: 各リージョン < 10ms
- **競合発生率**: 監視と対処

## パフォーマンス比較

| 構成 | 日本からのレイテンシ | シンガポールからのレイテンシ | 米国からのレイテンシ | コスト |
|------|---------------------|---------------------------|---------------------|--------|
| 単一（東京） | 5ms | 100ms | 200ms | $4 |
| グローバル | 5ms | 10ms | 150ms | $10 |
| グローバル（米含む） | 5ms | 10ms | 10ms | $15 |

## 学習リソース

- [DynamoDB Global Tables](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html)
- [Best Practices for Global Tables](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/V2globaltables_HowItWorks.html)
- [Global Tables Tutorial](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/V2globaltables.tutorial.html)

## 関連ドキュメント

- [データベース管理方針](../../docs/deploy/DATABASE.md)
- [デプロイ戦略](../../docs/deploy/STRATEGIES.md)

## 更新履歴

- 2024-12-21: 初版作成
