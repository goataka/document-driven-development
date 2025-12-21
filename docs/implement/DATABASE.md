# データベース設計

本ドキュメントは、DynamoDBをベースとしたデータベース設計の詳細を説明します。

**関連ドキュメント**: [システムアーキテクチャ設計書](../implement/)

## 目次

1. [テーブル設計](#テーブル設計)
2. [インデックス設計](#インデックス設計)
3. [データモデリング戦略](#データモデリング戦略)

---

## テーブル設計

### 1. Users テーブル

**パーティションキー**: `userId` (String, UUID)  
**ソートキー**: なし

| 属性名 | 型 | 説明 |
|---------|-----|-----|
| userId | String (UUID) | ユーザーID（PK） |
| email | String | メールアドレス |
| passwordHash | String | ハッシュ化パスワード |
| companyCode | String | 会社コード |
| createdAt | Number | 作成日時（Unix timestamp） |
| updatedAt | Number | 更新日時（Unix timestamp） |

**GSI**: `email-index`
- パーティションキー: `email`
- ソートキー: なし

**GSI**: `companyCode-index`
- パーティションキー: `companyCode`
- ソートキー: `createdAt`

### 2. Attendances テーブル

**パーティションキー**: `userId` (String)  
**ソートキー**: `clockInTime` (Number)

| 属性名 | 型 | 説明 |
|---------|-----|-----|
| userId | String (UUID) | ユーザーID（PK） |
| clockInTime | Number | 出勤時刻（SK, Unix timestamp） |
| attendanceId | String (UUID) | 勤怠記録ID |
| clockOutTime | Number | 退勤時刻（Unix timestamp） |
| workDurationMinutes | Number | 勤務時間（分） |
| createdAt | Number | 作成日時（Unix timestamp） |
| updatedAt | Number | 更新日時（Unix timestamp） |

---

## インデックス設計

### Users テーブルのインデックス

**email-index (GSI)**:
- 用途: メールアドレスでのユーザー検索
- プロジェクション: ALL

**companyCode-index (GSI)**:
- 用途: 会社コードでのユーザー一覧取得、作成日時順でソート
- プロジェクション: ALL

### Attendances テーブルのインデックス

**デフォルトインデックス**:
- パーティションキー: userId
- ソートキー: clockInTime
- 用途: ユーザーごとの勤怠履歴を時系列で取得

---

## データモデリング戦略

### アクセスパターン

1. **ユーザー認証**: email で検索 → email-index 使用
2. **会社別ユーザー一覧**: companyCode で検索 → companyCode-index 使用
3. **ユーザーの勤怠履歴**: userId で検索、clockInTime でソート → テーブルキー使用
4. **特定期間の勤怠**: userId + clockInTime の範囲クエリ → テーブルキー使用

### Single Table Design の検討

現在の設計では、Users と Attendances を別テーブルとしています。
今後のスケールに応じて、Single Table Design への移行を検討可能です。

詳細は[ADR-006: DynamoDB Single Table Design](../../specs/architecture/006-dynamodb-single-table-design.md)を参照してください。

### バックアップ戦略

- ポイントインタイムリカバリ（PITR）有効化
- 環境別バックアップ保持期間設定
- 定期的なオンデマンドバックアップ

---

**関連ドキュメント**:
- [システムアーキテクチャ設計書](../implement/)
- [バックエンド設計](./BACKEND.md)
- [API設計](./API.md)

---

**最終更新日**: 2024年12月21日  
**バージョン**: 2.0.0
