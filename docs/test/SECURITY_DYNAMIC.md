# 動的セキュリティテスト

本ドキュメントは、動的セキュリティテスト(OWASP ZAP、認証・認可、インジェクション攻撃)の実装方法を説明します。
**すべて無料のツールを使用しています。**

**関連ドキュメント**: 
- [テスト戦略概要](./)
- [静的セキュリティテスト](./SECURITY.md)
- [システムアーキテクチャ設計書](../implement/)
- [セキュリティ設計書](../implement/SECURITY.md)

## 目次

1. [動的セキュリティテスト](#動的セキュリティテスト)
2. [OWASP ZAP](#owasp-zap)
3. [認証・認可テスト](#認証認可テスト)
4. [インジェクション攻撃テスト](#インジェクション攻撃テスト)
5. [レート制限テスト](#レート制限テスト)
6. [ベストプラクティス](#ベストプラクティス)

---

## 動的セキュリティテスト

動的セキュリティテストは、実行中のアプリケーションに対して脆弱性を検出します。

## OWASP ZAP

OWASP ZAP（無料・オープンソース）を使用して動的セキュリティテストを実行します。

### Docker で実行

```bash
# OWASP ZAP のベースラインスキャン
docker run -t ghcr.io/zaproxy/zaproxy:stable zap-baseline.py \
  -t http://localhost:3000 \
  -r zap-report.html

# フルスキャン
docker run -t ghcr.io/zaproxy/zaproxy:stable zap-full-scan.py \
  -t http://localhost:3000 \
  -r zap-full-report.html
```

### CI/CD統合

```yaml
# .github/workflows/zap-scan.yml
name: OWASP ZAP Scan

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]
  schedule:
    # 毎週日曜日午前2時に実行
    - cron: '0 2 * * 0'

jobs:
  zap_scan:
    runs-on: ubuntu-latest
    name: OWASP ZAP Scan
    steps:
      - uses: actions/checkout@v4
      
      - name: Start application
        run: |
          docker-compose up -d
          sleep 30
      
      - name: ZAP Baseline Scan
        uses: zaproxy/action-baseline@v0.7.0
        with:
          target: 'http://localhost:3000'
          rules_file_name: '.zap/rules.tsv'
          cmd_options: '-a'
      
      - name: Stop application
        run: docker-compose down
```

---

## 認証・認可テスト

```typescript
// backend/test/security/auth.security.spec.ts
import { Test } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import * as request from 'supertest';
import { AppModule } from '../../src/app.module';

describe('認証・認可セキュリティテスト', () => {
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

  it('認証なしでは保護されたエンドポイントにアクセスできないこと', async () => {
    await request(app.getHttpServer())
      .get('/api/users')
      .expect(401);
  });

  it('無効なトークンでは認証できないこと', async () => {
    await request(app.getHttpServer())
      .get('/api/users')
      .set('Authorization', 'Bearer invalid-token')
      .expect(401);
  });

  it('有効期限切れのトークンでは認証できないこと', async () => {
    const expiredToken = 'expired-jwt-token';
    
    await request(app.getHttpServer())
      .get('/api/users')
      .set('Authorization', `Bearer ${expiredToken}`)
      .expect(401);
  });

  it('他のユーザーのデータにアクセスできないこと', async () => {
    const userAToken = 'valid-token-for-user-a';
    const userBId = 'user-b-id';
    
    await request(app.getHttpServer())
      .get(`/api/users/${userBId}`)
      .set('Authorization', `Bearer ${userAToken}`)
      .expect(403);
  });

  it('管理者権限が必要なエンドポイントは一般ユーザーがアクセスできないこと', async () => {
    const userToken = 'valid-token-for-regular-user';
    
    await request(app.getHttpServer())
      .post('/api/admin/users')
      .set('Authorization', `Bearer ${userToken}`)
      .send({ email: 'test@example.com', role: 'admin' })
      .expect(403);
  });
});
```

---

## インジェクション攻撃テスト

```typescript
// backend/test/security/injection.security.spec.ts
describe('インジェクション攻撃テスト', () => {
  it('SQLインジェクション攻撃を防ぐこと', async () => {
    const maliciousInput = "'; DROP TABLE users; --";
    
    await request(app.getHttpServer())
      .get('/api/users')
      .query({ search: maliciousInput })
      .set('Authorization', 'Bearer valid-token')
      .expect(200);
    
    // テーブルが削除されていないことを確認
    await request(app.getHttpServer())
      .get('/api/users')
      .set('Authorization', 'Bearer valid-token')
      .expect(200);
  });

  it('XSS攻撃を防ぐこと', async () => {
    const xssPayload = '<script>alert("XSS")</script>';
    
    const response = await request(app.getHttpServer())
      .post('/api/attendance/memo')
      .set('Authorization', 'Bearer valid-token')
      .send({ memo: xssPayload })
      .expect(201);
    
    // レスポンスでスクリプトがエスケープされていることを確認
    expect(response.body.memo).not.toContain('<script>');
    expect(response.body.memo).toContain('&lt;script&gt;');
  });

  it('コマンドインジェクション攻撃を防ぐこと', async () => {
    const maliciousFilename = 'file.txt; rm -rf /';
    
    await request(app.getHttpServer())
      .post('/api/export')
      .set('Authorization', 'Bearer valid-token')
      .send({ filename: maliciousFilename })
      .expect(400); // バリデーションエラー
  });
});
```

---

## レート制限テスト

```typescript
// backend/test/security/rate-limit.security.spec.ts
describe('レート制限テスト', () => {
  it('短時間に大量のリクエストが制限されること', async () => {
    const requests = Array(100).fill(null).map(() =>
      request(app.getHttpServer())
        .post('/api/auth/login')
        .send({ email: 'test@example.com', password: 'password' })
    );
    
    const responses = await Promise.all(requests);
    
    // 一部のリクエストが429 (Too Many Requests) であることを確認
    const tooManyRequestsCount = responses.filter(r => r.status === 429).length;
    expect(tooManyRequestsCount).toBeGreaterThan(0);
  });
});
```

---

## ベストプラクティス

1. **ペネトレーションテスト**
   - 本番前に実施
   - 第三者による監査も検討

2. **実際の攻撃パターンを再現**
   - OWASP Top 10を網羅
   - 新しい攻撃手法も追跡

3. **継続的なスキャン**
   - CI/CDで自動実行
   - 定期的なフルスキャン

4. **ログ監視**
   - 攻撃の兆候を検知
   - 異常なアクセスパターンを追跡

---

## 関連ドキュメント

- [テスト戦略概要](./)
- [静的セキュリティテスト](./SECURITY.md)
- [統合・E2Eテスト](./INTEGRATION.md)
- [セキュリティ設計書](../implement/SECURITY.md)
