# データベース管理

本ドキュメントは、データベースのマイグレーションとバックアップを説明します。

**関連ドキュメント**: 
- [デプロイ戦略概要](./README.md)
- [バックエンドデプロイ](./BACKEND.md)

## マイグレーション実行

```bash
# 本番環境でのマイグレーション
pg_dump -h localhost -U user attendance_db > backup.sql
npm run migration:run
npm run migration:show
```

---

**最終更新日**: 2024年12月19日  
**バージョン**: 1.0.0
