# 統合テスト

本ドキュメントは、統合テストの実装方法を説明します。

**関連ドキュメント**: 
- [テスト戦略概要](./OVERVIEW.md)
- [システムアーキテクチャ設計書](../implement/ARCHITECTURE.md)

## 目次

1. [統合テスト](#統合テスト)
3. [ベストプラクティス](#ベストプラクティス)

---

## 統合テスト

### バックエンド統合テスト: Jest + Supertest

**対象**:
- APIエンドポイントの動作確認
- 認証・認可フローの確認
- データベースとの連携確認

#### テスト例

```typescript
// test/attendance.e2e-spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication, ValidationPipe } from '@nestjs/common';
import * as request from 'supertest';
import { AppModule } from '../src/app.module';

describe('Attendance API (e2e)', () => {
  let app: INestApplication;
  let authToken: string;

  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    app.useGlobalPipes(new ValidationPipe());
    await app.init();

    // ログインしてトークン取得
    const loginResponse = await request(app.getHttpServer())
      .post('/api/auth/login')
      .send({
        email: 'test@example.com',
        password: 'password123',
      });
    authToken = loginResponse.body.token;
  });

  afterAll(async () => {
    await app.close();
  });

  describe('/api/attendance/clock-in (POST)', () => {
    it('出勤打刻が成功する', () => {
      return request(app.getHttpServer())
        .post('/api/attendance/clock-in')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(201)
        .expect((res) => {
          expect(res.body).toHaveProperty('id');
          expect(res.body).toHaveProperty('clockInTime');
          expect(res.body).toHaveProperty('clockOutTime', null);
        });
    });

    it('認証なしではエラーになる', () => {
      return request(app.getHttpServer())
        .post('/api/attendance/clock-in')
        .expect(401);
    });
  });

  describe('/api/attendance/history (GET)', () => {
    it('勤怠履歴を取得できる', () => {
      return request(app.getHttpServer())
        .get('/api/attendance/history')
        .query({ startDate: '2024-01-01', endDate: '2024-12-31' })
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200)
        .expect((res) => {
          expect(Array.isArray(res.body)).toBe(true);
        });
    });
  });
});
```

---



---


---


