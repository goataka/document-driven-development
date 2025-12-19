# バックアップとリストア

本ドキュメントは、バックアップ戦略とリストア手順を説明します。

**関連ドキュメント**: 
- [運用管理概要](./README.md)
- [メンテナンス](./MAINTENANCE.md)

## バックアップスクリプト

```bash
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups/database"
pg_dump -h localhost -U user -F c attendance_db > "$BACKUP_DIR/backup_$DATE.dump"
gzip "$BACKUP_DIR/backup_$DATE.dump"
```

## リストア手順

```bash
pg_restore -h localhost -U user -d attendance_db backup.dump
```

---

**最終更新日**: 2024年12月19日  
**バージョン**: 1.0.0
