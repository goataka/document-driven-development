# セキュリティ・脆弱性テスト

本ドキュメントは、セキュリティテストと脆弱性スキャンの実装方法を説明します。
**すべて無料のツールを使用しています。**

**関連ドキュメント**: 
- [テスト戦略概要](./TESTING.md)
- [システムアーキテクチャ設計書](./ARCHITECTURE.md)
- [セキュリティ設計書](./SECURITY.md)

## 目次

1. [セキュリティ・脆弱性テスト](#セキュリティ脆弱性テスト)
2. [ツール](#ツール)
3. [セキュリティテスト項目](#セキュリティテスト項目)
4. [セキュリティチェックリスト](#セキュリティチェックリスト)
5. [ベストプラクティス](#ベストプラクティス)

---

## セキュリティ・脆弱性テスト

セキュリティテストは、アプリケーションの脆弱性を検出し、セキュアな実装を保証します。
すべて無料のツールを使用します。

### npm audit

定期的に依存関係の脆弱性をチェックします。

```bash
# 脆弱性スキャン
npm audit

# 自動修正（メジャーバージョンは除く）
npm audit fix

# 全ての修正を適用
npm audit fix --force
```

### audit-ci

CI/CDで脆弱性チェックを強制するために、audit-ciを使用します（無料）。

#### インストールと設定

```bash
# audit-ci のインストール
npm install --save-dev audit-ci
```

#### package.json に追加

```json
{
  "scripts": {
    "audit:check": "audit-ci --moderate"
  }
}
```

#### CI/CD統合

```yaml
# .github/workflows/security.yml
name: Security Scan

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]
  schedule:
    # 毎日午前2時に実行
    - cron: '0 2 * * *'

jobs:
  security:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run npm audit
        run: npm audit --audit-level=moderate
      
      - name: Run audit-ci
        run: npx audit-ci --moderate
      
      - name: Dependency Review
        uses: actions/dependency-review-action@v4
        if: github.event_name == 'pull_request'
```

### GitHub CodeQL (無料)

GitHub の組み込みセキュリティスキャン機能を活用します。

```yaml
# .github/workflows/codeql.yml
name: CodeQL Analysis

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]
  schedule:
    - cron: '0 2 * * 1'

jobs:
  analyze:
    name: Analyze
    runs-on: ubuntu-latest
    permissions:
      actions: read
      contents: read
      security-events: write

    strategy:
      fail-fast: false
      matrix:
        language: ['javascript', 'typescript']

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}

      - name: Autobuild
        uses: github/codeql-action/autobuild@v3

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3
```

### OWASP ZAP (動的セキュリティテスト)

#### Docker で実行

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

### セキュリティテスト項目

#### 認証・認可テスト

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

#### インジェクション攻撃テスト

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

#### レート制限テスト

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

### セキュリティチェックリスト

- ✅ 全てのユーザー入力をバリデーション
- ✅ SQLインジェクション対策（パラメータ化クエリ使用）
- ✅ XSS対策（出力エスケープ）
- ✅ CSRF対策（トークン検証）
- ✅ 適切な認証・認可実装
- ✅ パスワードの安全なハッシュ化（bcrypt）
- ✅ HTTPS通信の強制
- ✅ セキュリティヘッダーの設定（Helmet使用）
- ✅ レート制限の実装
- ✅ 定期的な依存関係の更新と脆弱性スキャン

---



---

## ベストプラクティス

1. **セキュリティファースト**
   - 設計段階からセキュリティを考慮
   - 新機能ごとにセキュリティテスト

2. **定期的なスキャン**
   - 週次で脆弱性スキャン実行
   - 依存関係は常に最新に

3. **脆弱性は即修正**
   - Critical/High は即対応
   - Medium は1週間以内に対応

4. **ペネトレーションテスト**
   - 本番前に実施
   - 第三者による監査も検討

5. **セキュリティ教育**
   - チーム全体でセキュリティ意識向上
   - OWASPTop10を理解

---

## 関連ドキュメント

- [テスト戦略概要](./TESTING.md)
- [統合・E2Eテスト](./TESTING_INTEGRATION.md)
- [品質テスト](./TESTING_QUALITY.md)
- [依存関係管理と自動更新](./TESTING_DEPENDENCIES.md)
- [セキュリティ設計書](./SECURITY.md)
