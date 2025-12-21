# パフォーマンステスト

本ドキュメントは、パフォーマンステスト(Lighthouse CI、Core Web Vitals)の実装方法を説明します。

**関連ドキュメント**: 
- [テスト戦略概要](./)
- [システムアーキテクチャ設計書](../implement/)

## 目次

1. [パフォーマンステスト](#パフォーマンステスト)
2. [パフォーマンス目標](#パフォーマンス目標)
3. [ベストプラクティス](#ベストプラクティス)

---

## パフォーマンステスト

パフォーマンステストは、アプリケーションの応答速度と効率性を測定します。

### Lighthouse CI

#### インストールと設定

```bash
npm install --save-dev @lhci/cli
```

```javascript
// lighthouserc.js
module.exports = {
  ci: {
    collect: {
      url: ['http://localhost:3000/', 'http://localhost:3000/login'],
      numberOfRuns: 3,
      settings: {
        preset: 'desktop',
      },
    },
    assert: {
      preset: 'lighthouse:recommended',
      assertions: {
        'categories:performance': ['error', { minScore: 0.9 }],
        'categories:accessibility': ['error', { minScore: 0.9 }],
        'categories:best-practices': ['error', { minScore: 0.9 }],
        'categories:seo': ['error', { minScore: 0.9 }],
        // Core Web Vitals
        'first-contentful-paint': ['error', { maxNumericValue: 2000 }],
        'largest-contentful-paint': ['error', { maxNumericValue: 2500 }],
        'cumulative-layout-shift': ['error', { maxNumericValue: 0.1 }],
        'total-blocking-time': ['error', { maxNumericValue: 300 }],
      },
    },
    upload: {
      target: 'temporary-public-storage',
    },
  },
};
```

#### Lighthouse CI コマンド

```bash
# Lighthouse CI 実行
npm run lhci:collect
npm run lhci:assert

# package.json に追加
"scripts": {
  "lhci:collect": "lhci collect",
  "lhci:assert": "lhci assert",
  "lhci:upload": "lhci upload"
}
```

### Playwright パフォーマンステスト

```typescript
// e2e/tests/performance.spec.ts
import { test, expect } from '@playwright/test';

test.describe('パフォーマンステスト', () => {
  test('トップページが3秒以内に読み込まれること', async ({ page }) => {
    const startTime = Date.now();
    
    await page.goto('/', { waitUntil: 'networkidle' });
    
    const loadTime = Date.now() - startTime;
    expect(loadTime).toBeLessThan(3000);
  });

  test('ログイン処理が2秒以内に完了すること', async ({ page }) => {
    await page.goto('/login');
    
    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'password123');
    
    const startTime = Date.now();
    await page.click('button[type="submit"]');
    await page.waitForURL('/dashboard');
    
    const loginTime = Date.now() - startTime;
    expect(loginTime).toBeLessThan(2000);
  });

  test('大量データの表示が5秒以内に完了すること', async ({ page }) => {
    await page.goto('/attendance-history');
    
    const startTime = Date.now();
    
    // 最初の行と最後の行が表示されるまで待つ
    await page.waitForSelector('[data-testid="attendance-row"]:first-child');
    await page.waitForSelector('[data-testid="attendance-row"]:last-child');
    
    const renderTime = Date.now() - startTime;
    expect(renderTime).toBeLessThan(5000);
  });

  test('APIレスポンスが1秒以内であること', async ({ page }) => {
    await page.goto('/dashboard');
    
    // API リクエストを監視
    const responsePromise = page.waitForResponse(
      response => response.url().includes('/api/attendance') && response.status() === 200
    );
    
    const startTime = Date.now();
    await page.click('[data-testid="refresh-button"]');
    const response = await responsePromise;
    const responseTime = Date.now() - startTime;
    
    expect(responseTime).toBeLessThan(1000);
    expect(response.status()).toBe(200);
  });
});
```

### バックエンドパフォーマンステスト

```typescript
// backend/test/performance/api.performance.spec.ts
import { Test } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import * as request from 'supertest';
import { AppModule } from '../../src/app.module';

describe('API パフォーマンステスト', () => {
  let app: INestApplication;

  beforeAll(async () => {
    const moduleRef = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleRef.createNestApplication();
    await app.init();
  });

  afterAll(async () => {
    await app.close();
  });

  it('GET /api/users が100ms以内にレスポンスすること', async () => {
    const startTime = Date.now();
    
    const response = await request(app.getHttpServer())
      .get('/api/users')
      .set('Authorization', 'Bearer valid-token')
      .expect(200);
    
    const responseTime = Date.now() - startTime;
    expect(responseTime).toBeLessThan(100);
    expect(response.body).toBeDefined();
  });

  it('GET /api/attendance が200ms以内にレスポンスすること', async () => {
    const startTime = Date.now();
    
    const response = await request(app.getHttpServer())
      .get('/api/attendance?limit=100')
      .set('Authorization', 'Bearer valid-token')
      .expect(200);
    
    const responseTime = Date.now() - startTime;
    expect(responseTime).toBeLessThan(200);
    expect(response.body.data).toHaveLength(100);
  });

  it('POST /api/attendance/clock-in が150ms以内にレスポンスすること', async () => {
    const startTime = Date.now();
    
    const response = await request(app.getHttpServer())
      .post('/api/attendance/clock-in')
      .set('Authorization', 'Bearer valid-token')
      .expect(201);
    
    const responseTime = Date.now() - startTime;
    expect(responseTime).toBeLessThan(150);
    expect(response.body.id).toBeDefined();
  });
});
```

---

## パフォーマンス目標

| メトリクス | 目標値 |
|----------|-------|
| First Contentful Paint (FCP) | < 2.0秒 |
| Largest Contentful Paint (LCP) | < 2.5秒 |
| Total Blocking Time (TBT) | < 300ms |
| Cumulative Layout Shift (CLS) | < 0.1 |
| Time to Interactive (TTI) | < 3.5秒 |
| API レスポンス時間 | < 200ms (P95) |

---

## ベストプラクティス

1. **定期的に測定**
   - CI/CDで自動実行
   - 劣化を早期検知

2. **実際のデータ量で測定**
   - 本番相当のデータ
   - 負荷がかかる状況

3. **ボトルネックを特定**
   - プロファイリングツール活用
   - データベースクエリ最適化

---
