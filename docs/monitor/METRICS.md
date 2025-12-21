# メトリクス収集

本ドキュメントは、Prometheusメトリクス収集を説明します。

## Prometheus設定

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'attendance-backend'
    static_configs:
      - targets: ['localhost:3000']
```

## カスタムメトリクス

```typescript
import { Counter, Histogram } from 'prom-client';

export const httpRequestCounter = new Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});
```

---

**最終更新日**: 2024年12月19日  
**バージョン**: 1.0.0
