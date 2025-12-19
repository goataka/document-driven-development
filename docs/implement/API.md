# API設計

本ドキュメントは、RESTful APIの設計詳細を説明します。

**関連ドキュメント**: [システムアーキテクチャ設計書](../implement/)

## 目次

1. [RESTful API原則](#restful-api原則)
2. [エンドポイント一覧](#エンドポイント一覧)
3. [レスポンス形式](#レスポンス形式)
4. [APIドキュメント](#apiドキュメント)

---

## RESTful API原則

- **リソース指向**: URL設計はリソースを表現
- **HTTPメソッド**: GET, POST, PUT/PATCH, DELETEを適切に使用
- **ステータスコード**: 適切なHTTPステータスコードを返却
- **JSON形式**: リクエスト・レスポンスはJSON

---

## エンドポイント一覧

### 認証API

| メソッド | エンドポイント | 説明 | 認証 |
|---------|--------------|------|-----|
| POST | /api/auth/register | ユーザー登録 | 不要 |
| POST | /api/auth/login | ログイン | 不要 |
| POST | /api/auth/logout | ログアウト | 必要 |
| GET | /api/auth/me | 現在のユーザー情報取得 | 必要 |

### ユーザーAPI

| メソッド | エンドポイント | 説明 | 認証 |
|---------|--------------|------|-----|
| GET | /api/users/profile | プロフィール取得 | 必要 |
| PUT | /api/users/profile | プロフィール更新 | 必要 |

### 勤怠API

| メソッド | エンドポイント | 説明 | 認証 |
|---------|--------------|------|-----|
| POST | /api/attendance/clock-in | 出勤打刻 | 必要 |
| POST | /api/attendance/clock-out | 退勤打刻 | 必要 |
| GET | /api/attendance/history | 勤怠履歴取得 | 必要 |
| GET | /api/attendance/today | 本日の打刻状況 | 必要 |

---

## レスポンス形式

### 成功時

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "clockInTime": "2024-01-01T09:00:00Z",
    "clockOutTime": null
  }
}
```

### エラー時

```json
{
  "success": false,
  "error": {
    "code": "ALREADY_CLOCKED_IN",
    "message": "既に出勤打刻済みです",
    "details": {}
  }
}
```

---

## APIドキュメント

- **Swagger UI**を使用してAPIドキュメントを自動生成
- エンドポイント: `/api/docs`
- 全APIエンドポイントの仕様、リクエスト/レスポンス例を記載

---

**関連ドキュメント**:
- [システムアーキテクチャ設計書](../implement/)
- [バックエンド設計](./BACKEND.md)
- [セキュリティ](./SECURITY.md)

---

**最終更新日**: 2024年12月17日  
**バージョン**: 1.0.0
