# 統合・E2Eテスト

本ドキュメントは、統合テストとE2Eテストの実装方法を説明します。

**関連ドキュメント**: 
- [テスト戦略概要](./TESTING.md)
- [システムアーキテクチャ設計書](../spec/ARCHITECTURE.md)

## 目次

1. [統合テスト](#統合テスト)
2. [E2Eテスト (Cucumber + Playwright)](#e2eテスト-cucumber--playwright)
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

## E2Eテスト (Cucumber + Playwright)

**目的**:
- ユーザーシナリオ全体の動作確認
- ブラウザでの実際の挙動確認
- ビジネス要件の検証

**技術スタック**:
- **Cucumber**: BDD（振る舞い駆動開発）フレームワーク、Gherkin記法
- **Playwright**: クロスブラウザ自動化ツール

### ディレクトリ構成

```
e2e/
├── features/                    # Gherkin feature files
│   ├── login.feature
│   ├── clock-in-out.feature
│   └── attendance-history.feature
├── step-definitions/            # ステップ定義
│   ├── auth.steps.ts
│   ├── attendance.steps.ts
│   └── common.steps.ts
├── support/                     # ヘルパー
│   ├── world.ts
│   ├── hooks.ts
│   └── page-objects/
│       ├── LoginPage.ts
│       ├── DashboardPage.ts
│       └── AttendanceHistoryPage.ts
├── cucumber.js                  # Cucumber設定
└── playwright.config.ts         # Playwright設定
```

### Feature例

```gherkin
# e2e/features/login.feature
# language: ja
機能: ログイン機能
  ユーザーとして
  システムにログインしたい
  勤怠管理機能を利用するため

  背景:
    前提 ユーザー "user@example.com" がパスワード "password123" で登録されている

  シナリオ: 正しい認証情報でログインする
    前提 ログインページを表示している
    もし メールアドレス "user@example.com" を入力する
    かつ パスワード "password123" を入力する
    かつ "ログイン" ボタンをクリックする
    ならば ダッシュボードページが表示される
    かつ ユーザー名が表示される

  シナリオ: 間違ったパスワードでログインを試みる
    前提 ログインページを表示している
    もし メールアドレス "user@example.com" を入力する
    かつ パスワード "wrongpassword" を入力する
    かつ "ログイン" ボタンをクリックする
    ならば エラーメッセージ "メールアドレスまたはパスワードが正しくありません" が表示される
    かつ ログインページに留まる
```

### ステップ定義例

```typescript
// e2e/step-definitions/auth.steps.ts
import { Given, When, Then } from '@cucumber/cucumber';
import { expect } from '@playwright/test';
import { ICustomWorld } from '../support/world';
import { LoginPage } from '../support/page-objects/LoginPage';

// ヘルパー関数でnullチェックを集約
function ensurePage(world: ICustomWorld) {
  if (!world.page) throw new Error('Page not initialized');
  return world.page;
}

function ensureLoginPage(world: ICustomWorld) {
  if (!world.loginPage) throw new Error('Login page not initialized');
  return world.loginPage;
}

Given('ログインページを表示している', async function (this: ICustomWorld) {
  const page = ensurePage(this);
  this.loginPage = new LoginPage(page);
  await this.loginPage.goto();
});

When('メールアドレス {string} を入力する', async function (this: ICustomWorld, email: string) {
  const loginPage = ensureLoginPage(this);
  await loginPage.fillEmail(email);
});

When('パスワード {string} を入力する', async function (this: ICustomWorld, password: string) {
  const loginPage = ensureLoginPage(this);
  await loginPage.fillPassword(password);
});

When('{string} ボタンをクリックする', async function (this: ICustomWorld, buttonText: string) {
  const page = ensurePage(this);
  // data-testid属性を優先（より安全）、フォールバックとしてテキスト検索
  const testIdSelector = `[data-testid="${buttonText}-button"]`;
  const textSelector = `button:has-text("${buttonText}")`;
  await page.click(`${testIdSelector}, ${textSelector}`);
});

Then('ダッシュボードページが表示される', async function (this: ICustomWorld) {
  const page = ensurePage(this);
  await expect(page).toHaveURL(/.*dashboard/);
});

Then('エラーメッセージ {string} が表示される', async function (this: ICustomWorld, message: string) {
  const page = ensurePage(this);
  // data-testid属性を使用してより安定したセレクタに
  await expect(page.locator('[data-testid="error-message"]')).toContainText(message);
});
```

### Page Object例

```typescript
// e2e/support/page-objects/LoginPage.ts
import { Page, Locator } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly loginButton: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.locator('input[name="email"]');
    this.passwordInput = page.locator('input[name="password"]');
    this.loginButton = page.locator('button[type="submit"]');
  }

  async goto() {
    await this.page.goto('/login');
  }

  async fillEmail(email: string) {
    await this.emailInput.fill(email);
  }

  async fillPassword(password: string) {
    await this.passwordInput.fill(password);
  }

  async clickLogin() {
    await this.loginButton.click();
  }

  async login(email: string, password: string) {
    await this.fillEmail(email);
    await this.fillPassword(password);
    await this.clickLogin();
  }
}
```

### Playwright設定

```typescript
// e2e/playwright.config.ts
import { PlaywrightTestConfig } from '@playwright/test';

const config: PlaywrightTestConfig = {
  testDir: './e2e',
  timeout: 30000,
  retries: 2,
  use: {
    baseURL: 'http://localhost:5173',
    headless: true,
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
  projects: [
    {
      name: 'chromium',
      use: { browserName: 'chromium' },
    },
    {
      name: 'firefox',
      use: { browserName: 'firefox' },
    },
    {
      name: 'webkit',
      use: { browserName: 'webkit' },
    },
  ],
};

export default config;
```

**実行コマンド**:

```bash
# E2Eテストの実行
npm run test:e2e

# 特定のfeatureのみ実行
npm run test:e2e -- --name="ログイン機能"

# ヘッドレスモードをオフにして実行
npm run test:e2e -- --headed

# 特定のブラウザで実行
npm run test:e2e -- --project=chromium
```

---



---

## ベストプラクティス

### 統合テスト

1. **実際のデータベースを使用**
   - テスト用データベースを用意
   - トランザクションでロールバック

2. **APIレベルでテスト**
   - コントローラーからエンドポイントまで
   - 実際のHTTPリクエスト/レスポンス

3. **認証を含める**
   - JWT トークンの生成・検証
   - 権限チェック

### E2Eテスト

1. **重要なユーザーフローに集中**
   - 全機能ではなく、主要なシナリオ
   - ビジネスクリティカルな機能優先

2. **Page Objectパターンを使用**
   - UI要素の変更に強い
   - 再利用性が高い

3. **待機を適切に設定**
   - 固定時間ではなく要素の出現を待つ
   - タイムアウトは適切に設定

4. **テストデータを独立させる**
   - 各テストで独自のデータを作成
   - クリーンアップを適切に実施

---

## 関連ドキュメント

- [テスト戦略概要](./TESTING.md)
- [スナップショットテスト](./TESTING_SNAPSHOT.md)
- [アクセシビリティテスト](./TESTING_ACCESSIBILITY.md)
- [パフォーマンステスト](./TESTING_PERFORMANCE.md)
- [静的セキュリティテスト](./TESTING_SECURITY.md)
- [動的セキュリティテスト](./TESTING_SECURITY_DYNAMIC.md)
