# システムメンテナンス

本ドキュメントは、定期メンテナンスを説明します。

**関連ドキュメント**: 
- [運用管理概要](./README.md)
- [日常運用](./DAILY.md)

## データベースメンテナンス

```bash
# VACUUM実行（週次）
psql -U user -d attendance_db -c "VACUUM ANALYZE;"

# インデックスの再構築（月次）
psql -U user -d attendance_db -c "REINDEX DATABASE attendance_db;"
```

---

**最終更新日**: 2024年12月19日  
**バージョン**: 1.0.0
