# エラー追跡

本ドキュメントは、Sentryエラー追跡を説明します。

**関連ドキュメント**: 
- [監視戦略概要](./README.md)
- [ログ管理](./LOGGING.md)

## Sentry設定

### バックエンド

```typescript
import * as Sentry from '@sentry/node';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 0.1
});
```

### フロントエンド

```typescript
import * as Sentry from '@sentry/react';

Sentry.init({
  dsn: import.meta.env.VITE_SENTRY_DSN,
  integrations: [new Sentry.BrowserTracing()],
  tracesSampleRate: 0.1
});
```

---

**最終更新日**: 2024年12月19日  
**バージョン**: 1.0.0
