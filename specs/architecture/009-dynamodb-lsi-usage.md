# ADR-009: DynamoDB Local Secondary Index（LSI）の利用

## ステータス

提案中（Proposed）

## 背景

DynamoDBでは、パーティションキーとソートキー以外の属性でクエリを実行するため、セカンダリインデックスの利用を検討する必要があります。

### セカンダリインデックスの種類

1. **Global Secondary Index (GSI)**: 別のパーティションキーとソートキーを使用
2. **Local Secondary Index (LSI)**: 同じパーティションキー、異なるソートキーを使用

## 決定事項

**検討中**: クエリパターンに応じて、LSI の必要性を判断する。

### LSI を使用すべきケース

1. **同一パーティション内の異なるソート**: 同じユーザーのデータを異なる順序で取得
2. **強い整合性が必要**: 結果整合性ではなく、強い整合性が必要
3. **追加コストを避けたい**: GSIと異なり、追加の容量単位が不要

### GSI を使用すべきケース

1. **異なるパーティションキー**: 完全に異なるアクセスパターン
2. **作成後の追加**: テーブル作成後でも追加可能
3. **スパースインデックス**: 一部のアイテムのみインデックス化

## 現在のデータモデル

### Attendances テーブル

```
PK (Partition Key): userId
SK (Sort Key): clockInTime
属性: clockOutTime, workHours, status, date
```

### 現在のアクセスパターン

1. **ユーザーの勤怠履歴（時系列）**: 
   - クエリ: `userId` でフィルタ、`clockInTime` でソート
   - 実装: テーブルキーで対応可能 ✅

2. **ユーザーの勤怠履歴（日付順）**:
   - クエリ: `userId` でフィルタ、`date` でソート
   - 実装: LSI が有効 ⚠️

3. **特定ステータスの勤怠**:
   - クエリ: `userId` でフィルタ、`status` でフィルタ
   - 実装: FilterExpression または GSI

## 結果

### LSI の実装例

#### テーブル作成時にLSIを定義

```typescript
import { CreateTableCommand } from '@aws-sdk/client-dynamodb';

const command = new CreateTableCommand({
  TableName: 'attendances',
  KeySchema: [
    { AttributeName: 'userId', KeyType: 'HASH' },
    { AttributeName: 'clockInTime', KeyType: 'RANGE' },
  ],
  AttributeDefinitions: [
    { AttributeName: 'userId', AttributeType: 'S' },
    { AttributeName: 'clockInTime', AttributeType: 'S' },
    { AttributeName: 'date', AttributeType: 'S' }, // LSI用
  ],
  LocalSecondaryIndexes: [
    {
      IndexName: 'userId-date-index',
      KeySchema: [
        { AttributeName: 'userId', KeyType: 'HASH' },
        { AttributeName: 'date', KeyType: 'RANGE' },
      ],
      Projection: {
        ProjectionType: 'ALL', // すべての属性を含める
      },
    },
  ],
  BillingMode: 'PAY_PER_REQUEST',
});
```

#### LSI を使用したクエリ

```typescript
import { QueryCommand } from '@aws-sdk/lib-dynamodb';

// 日付順で勤怠を取得
const result = await docClient.send(
  new QueryCommand({
    TableName: 'attendances',
    IndexName: 'userId-date-index',
    KeyConditionExpression: 'userId = :userId AND #date BETWEEN :startDate AND :endDate',
    ExpressionAttributeNames: {
      '#date': 'date',
    },
    ExpressionAttributeValues: {
      ':userId': '123',
      ':startDate': '2024-01-01',
      ':endDate': '2024-01-31',
    },
  })
);
```

### メリット（LSI）

- **強い整合性**: 即座に最新データを取得
- **追加コストなし**: テーブルと同じ容量単位を使用
- **同一パーティション**: 効率的なクエリ
- **複数ソートキー**: 異なる順序でのアクセスが可能

### デメリット（LSI）

- **テーブル作成時のみ**: 後から追加できない ⚠️
- **最大5個**: テーブルあたり5個まで
- **パーティションキーは同じ**: 柔軟性が限定的
- **アイテムサイズ制限**: パーティション内の合計サイズ制限（10GB）

## 代替案

### 1. FilterExpression を使用

```typescript
// LSIなしでdate順に取得（全データをスキャン後フィルタ）
const result = await docClient.send(
  new QueryCommand({
    TableName: 'attendances',
    KeyConditionExpression: 'userId = :userId',
    FilterExpression: '#date BETWEEN :startDate AND :endDate',
    ExpressionAttributeNames: {
      '#date': 'date',
    },
    ExpressionAttributeValues: {
      ':userId': '123',
      ':startDate': '2024-01-01',
      ':endDate': '2024-01-31',
    },
  })
);
```

**デメリット**: すべてのアイテムを読み取ってからフィルタ（非効率、コスト増）

### 2. GSI を使用

```typescript
// GSIを作成（テーブル作成後でも可能）
GlobalSecondaryIndexes: [
  {
    IndexName: 'date-index',
    KeySchema: [
      { AttributeName: 'date', KeyType: 'HASH' },
      { AttributeName: 'userId', KeyType: 'RANGE' },
    ],
    Projection: {
      ProjectionType: 'ALL',
    },
  },
]
```

**デメリット**: 追加の容量単位が必要（コスト増）

### 3. アプリケーションレベルでソート

```typescript
// すべてのデータを取得後、アプリケーションでソート
const result = await docClient.send(
  new QueryCommand({
    TableName: 'attendances',
    KeyConditionExpression: 'userId = :userId',
    ExpressionAttributeValues: { ':userId': '123' },
  })
);

const sortedByDate = result.Items.sort((a, b) => 
  a.date.localeCompare(b.date)
);
```

**デメリット**: 大量データでメモリ使用量増加、パフォーマンス低下

## 推奨アプローチ

### 現在の要件

勤怠管理システムでは以下のクエリパターンが想定される：

1. ✅ **時系列順の勤怠取得**: テーブルキー（`clockInTime`）で対応
2. ⚠️ **日付順の勤怠取得**: LSI（`date`）が有効だが、必須ではない
3. ✅ **特定期間の勤怠**: テーブルキーで対応可能（`clockInTime`の範囲クエリ）

### 推奨

**初期実装ではLSIなし**、必要に応じてテーブル再作成時にLSIを追加する。

理由：
- `clockInTime` と `date` は通常同じ順序
- 特定期間のクエリはテーブルキーで対応可能
- 後から追加できないため、慎重に判断

## パフォーマンス比較

| 方法 | レイテンシ | コスト | 柔軟性 |
|------|-----------|--------|--------|
| テーブルキー | 5-10ms | 低 | 低 |
| LSI | 5-10ms | 低（追加なし） | 中 |
| GSI | 5-10ms | 中（追加容量） | 高 |
| FilterExpression | 20-50ms | 高（全読み取り） | 高 |
| アプリソート | 50-200ms | 中 | 高 |

## 学習リソース

- [DynamoDB Secondary Indexes](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/SecondaryIndexes.html)
- [Best Practices for Using Secondary Indexes](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-indexes.html)
- [LSI vs GSI](https://aws.amazon.com/blogs/database/choosing-the-right-dynamodb-partition-key/)

## 関連ドキュメント

- [データベース設計](../../docs/implement/DATABASE.md)
- [データベース管理方針](../../docs/deploy/DATABASE.md)

## 更新履歴

- 2024-12-21: 初版作成
