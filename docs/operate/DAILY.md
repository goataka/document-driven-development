# 日常運用タスク

本ドキュメントは、日常的な運用タスクを説明します。

## 毎日のチェックリスト

### 朝（9:00）
```bash
# システムヘルスチェック
curl https://api.example.com/health

# ログ確認
tail -n 100 /var/log/application.log | grep -i error
```

### 夕方（18:00）
```bash
# エラー集計
grep -c "ERROR" /var/log/application.log
```

---

**最終更新日**: 2024年12月19日  
**バージョン**: 1.0.0
