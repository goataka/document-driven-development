# ログ管理

本ドキュメントは、ログ管理とロガー設定を説明します。

**関連ドキュメント**: 
- [監視戦略概要](./README.md)
- [メトリクス収集](./METRICS.md)

## ログレベル

| レベル | 用途 |
|-------|------|
| ERROR | エラー、例外 |
| WARN | 警告 |
| INFO | 情報 |
| DEBUG | デバッグ |

## Winston設定

```typescript
import * as winston from 'winston';

export const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'error.log', level: 'error' })
  ]
});
```

---

**最終更新日**: 2024年12月19日  
**バージョン**: 1.0.0
