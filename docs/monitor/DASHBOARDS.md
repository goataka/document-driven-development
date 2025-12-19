# ダッシュボード

本ドキュメントは、Grafanaダッシュボード設定を説明します。

**関連ドキュメント**: 
- [監視戦略概要](./README.md)
- [メトリクス収集](./METRICS.md)

## Grafanaダッシュボード

### システム概要

- Request Rate
- Response Time (95th percentile)
- Error Rate
- Active Users

## ダッシュボード設定

```json
{
  "dashboard": {
    "title": "Attendance System Overview",
    "panels": [
      {
        "title": "Request Rate",
        "targets": [{"expr": "rate(http_requests_total[5m])"}]
      }
    ]
  }
}
```

---

**最終更新日**: 2024年12月19日  
**バージョン**: 1.0.0
