# データベース設計

本ドキュメントは、PostgreSQLをベースとしたデータベース設計の詳細を説明します。

**関連ドキュメント**: [システムアーキテクチャ設計書](../implement/ARCHITECTURE.md)

## 目次

1. [テーブル設計](#テーブル設計)
2. [インデックス設計](#インデックス設計)
3. [マイグレーション戦略](#マイグレーション戦略)

---

## テーブル設計

### 1. users テーブル

| カラム名 | 型 | 制約 | 説明 |
|---------|-----|-----|-----|
| id | UUID | PRIMARY KEY | ユーザーID |
| email | VARCHAR(255) | UNIQUE, NOT NULL | メールアドレス |
| password | VARCHAR(255) | NOT NULL | ハッシュ化パスワード |
| company_code | VARCHAR(50) | NOT NULL | 会社コード |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 作成日時 |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 更新日時 |

### 2. attendances テーブル

| カラム名 | 型 | 制約 | 説明 |
|---------|-----|-----|-----|
| id | UUID | PRIMARY KEY | 勤怠記録ID |
| user_id | UUID | FOREIGN KEY (users.id), NOT NULL | ユーザーID |
| clock_in_time | TIMESTAMP | NOT NULL | 出勤時刻 |
| clock_out_time | TIMESTAMP | NULL | 退勤時刻 |
| work_duration_minutes | INTEGER | NULL | 勤務時間（分） |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 作成日時 |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 更新日時 |

---

## インデックス設計

```sql
-- users テーブル
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_company_code ON users(company_code);

-- attendances テーブル
CREATE INDEX idx_attendances_user_id ON attendances(user_id);
CREATE INDEX idx_attendances_clock_in_time ON attendances(clock_in_time);
CREATE INDEX idx_attendances_user_clock_in ON attendances(user_id, clock_in_time);
```

---

## マイグレーション戦略

- **TypeORM Migrations**または**Prisma Migrate**を使用
- 本番環境へのマイグレーションは自動化せず、手動実行を推奨
- ロールバック計画を事前に準備
- バックアップを必ず取得してから実行

---

**関連ドキュメント**:
- [システムアーキテクチャ設計書](../implement/ARCHITECTURE.md)
- [バックエンド設計](./BACKEND.md)
- [API設計](./API.md)

---

**最終更新日**: 2024年12月17日  
**バージョン**: 1.0.0
