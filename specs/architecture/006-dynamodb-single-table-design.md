# ADR-006: DynamoDB Single Table Designの採用

## ステータス

提案中（Proposed）

## 背景

現在の設計では、Users と Attendances を別テーブルとして管理しています。DynamoDBのベストプラクティスとして、Single Table Design（単一テーブル設計）が推奨されるケースがあります。

### Single Table Design とは

異なるエンティティタイプを1つのテーブルに格納し、パーティションキーとソートキーを工夫することで、効率的なクエリを実現する設計手法です。

### 現在の設計（Multiple Table Design）

```
テーブル: users
PK: userId
属性: email, name, password, companyCode

テーブル: attendances
PK: userId
SK: clockInTime
属性: clockOutTime, workHours, status
```

### Single Table Design の例

```
テーブル: app-data
PK              | SK                    | Type       | 属性
----------------|----------------------|------------|------------------
USER#123        | PROFILE              | User       | email, name...
USER#123        | ATTENDANCE#2024-01-01| Attendance | clockIn, clockOut...
COMPANY#ABC     | USER#123             | UserIndex  | userId, name...
EMAIL#user@example.com | PROFILE     | EmailIndex | userId
```

## 決定事項

**推奨**: 初期実装では Multiple Table Design を採用し、スケール要件に応じて Single Table Design への移行を検討する。

### Multiple Table Design を初期採用する理由

1. **シンプルさ**: 理解しやすく、実装が容易
2. **柔軟性**: テーブルごとの独立した管理
3. **学習コスト**: DynamoDBの知識が少なくても対応可能
4. **デバッグ**: データ構造が直感的で確認しやすい

### Single Table Design を検討すべきケース

1. **高トラフィック**: 秒間1000リクエスト以上
2. **コスト最適化**: リクエスト数削減が重要
3. **複雑なクエリパターン**: 多数のアクセスパターンを効率化したい
4. **トランザクション**: 複数エンティティの原子的更新が必要

## 結果

### メリット（Multiple Table Design）

- 実装が簡単
- テーブルごとの独立したスケーリング
- 将来的な変更が容易
- バックアップ・リストアが単純

### デメリット

- 複数テーブルへのアクセスが必要
- リクエスト数の増加（コスト増）
- トランザクション対応が複雑

### メリット（Single Table Design）

- リクエスト数の削減（コスト削減）
- 一貫性の保証が容易（トランザクション）
- 複雑なクエリパターンの最適化
- DynamoDBのベストプラクティスに準拠

### デメリット

- 設計の複雑さ
- データモデリングの学習コスト
- デバッグの難しさ
- スキーマ変更の影響が大きい

## 移行戦略

### Phase 1（現在）: Multiple Table Design
- Users テーブル
- Attendances テーブル

### Phase 2（将来）: ハイブリッド構成
- 頻繁にアクセスするデータを Single Table に移行
- その他は Multiple Table のまま

### Phase 3（長期）: Full Single Table Design
- すべてのエンティティを1つのテーブルに統合
- アクセスパターンの最適化

## 実装例

### Multiple Table Design（現在）

```typescript
// Users テーブルへのアクセス
const user = await docClient.send(
  new GetCommand({
    TableName: 'users',
    Key: { userId: '123' },
  })
);

// Attendances テーブルへのアクセス
const attendances = await docClient.send(
  new QueryCommand({
    TableName: 'attendances',
    KeyConditionExpression: 'userId = :userId',
    ExpressionAttributeValues: { ':userId': '123' },
  })
);
```

### Single Table Design（将来）

```typescript
// ユーザープロファイルの取得
const user = await docClient.send(
  new GetCommand({
    TableName: 'app-data',
    Key: { 
      PK: 'USER#123', 
      SK: 'PROFILE' 
    },
  })
);

// ユーザーの勤怠記録を一度に取得
const userWithAttendances = await docClient.send(
  new QueryCommand({
    TableName: 'app-data',
    KeyConditionExpression: 'PK = :pk AND begins_with(SK, :sk)',
    ExpressionAttributeValues: {
      ':pk': 'USER#123',
      ':sk': 'ATTENDANCE#',
    },
  })
);
```

## パフォーマンス比較

### Multiple Table Design
- 2回のリクエスト（User + Attendances）
- レイテンシ: 10-20ms（合計）
- コスト: 2 Read Capacity Units

### Single Table Design
- 1回のクエリで取得
- レイテンシ: 5-10ms
- コスト: 1 Read Capacity Unit

## 学習リソース

- [AWS DynamoDB Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)
- [The DynamoDB Book by Alex DeBrie](https://www.dynamodbbook.com/)
- [AWS re:Invent - Advanced Design Patterns](https://www.youtube.com/watch?v=HaEPXoXVf2k)

## 関連ドキュメント

- [データベース設計](../../docs/implement/DATABASE.md)
- [データベース管理方針](../../docs/deploy/DATABASE.md)

## 更新履歴

- 2024-12-21: 初版作成
