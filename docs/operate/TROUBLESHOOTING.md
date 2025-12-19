# トラブルシューティング

本ドキュメントは、一般的な問題と解決方法を説明します。

**関連ドキュメント**: 
- [運用管理概要](./README.md)
- [メンテナンス](./MAINTENANCE.md)

## よくある問題

### 1. アプリケーションが応答しない
```bash
systemctl status attendance-backend
systemctl restart attendance-backend
```

### 2. データベース接続エラー
```bash
psql -U user -d attendance_db -c "SELECT count(*) FROM pg_stat_activity;"
```

---

**最終更新日**: 2024年12月19日  
**バージョン**: 1.0.0
