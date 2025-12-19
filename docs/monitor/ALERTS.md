# アラート設定

本ドキュメントは、Alertmanagerアラート設定を説明します。

**関連ドキュメント**: 
- [監視戦略概要](./README.md)
- [メトリクス収集](./METRICS.md)

## アラートルール

```yaml
groups:
  - name: attendance_alerts
    rules:
      - alert: HighResponseTime
        expr: http_request_duration_seconds{quantile="0.95"} > 5
        for: 5m
        labels:
          severity: warning
```

## Alertmanager設定

```yaml
route:
  group_by: ['alertname', 'severity']
  receiver: 'default'

receivers:
  - name: 'default'
    email_configs:
      - to: 'ops-team@example.com'
```

---

**最終更新日**: 2024年12月19日  
**バージョン**: 1.0.0
