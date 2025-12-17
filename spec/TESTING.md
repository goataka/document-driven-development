# テスト戦略

本ドキュメントは、システム全体のテスト戦略と実装方法の詳細を説明します。

**関連ドキュメント**: [システムアーキテクチャ設計書](./ARCHITECTURE.md)

## 目次

1. [テストピラミッド](#テストピラミッド)
2. [単体テスト](#単体テスト)
3. [コンポーネントテスト (Storybook)](#コンポーネントテスト-storybook)
4. [統合テスト](#統合テスト)
5. [E2Eテスト (Cucumber + Playwright)](#e2eテスト-cucumber--playwright)
6. [テストカバレッジ目標](#テストカバレッジ目標)
7. [CI/CD統合](#cicd統合)
8. [ベストプラクティス](#ベストプラクティス)

---

## テストピラミッド

テストは品質保証の要であり、以下の複数のレイヤーで包括的にテストを実施します。

```
           ┌─────────────────┐
           │   E2Eテスト     │  少数・遅い・高コスト
           │   (Cucumber +   │
           │   Playwright)   │
           └─────────────────┘
                   △
                  ╱ ╲
                 ╱   ╲
                ╱     ╲
               ╱       ╲
         ┌────────────────┐
         │  統合テスト     │   中程度
         │  (Jest)        │
         └────────────────┘
                △
               ╱ ╲
              ╱   ╲
             ╱     ╲
            ╱       ╲
      ┌──────────────────┐
      │   単体テスト      │   多数・速い・低コスト
      │   (Jest/Vitest)  │
      └──────────────────┘
```

---

## 単体テスト

### フロントエンド: Vitest + React Testing Library

**対象**:
- ユーティリティ関数
- カスタムフック
- 状態管理ロジック
- 個別のReactコンポーネント

#### Vitest設定

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './src/test/setup.ts',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'src/test/',
      ]
    }
  }
});
```

#### フックテスト例

```typescript
// src/hooks/__tests__/useAuth.test.ts
import { renderHook, act } from '@testing-library/react';
import { describe, it, expect, beforeEach } from 'vitest';
import { useAuthStore } from '../useAuth';

describe('useAuth', () => {
  beforeEach(() => {
    // ストアをリセット
    useAuthStore.getState().logout();
  });

  it('初期状態では認証されていない', () => {
    const { result } = renderHook(() => useAuthStore());
    expect(result.current.isAuthenticated).toBe(false);
    expect(result.current.user).toBeNull();
  });

  it('ログイン後は認証状態になる', async () => {
    const { result } = renderHook(() => useAuthStore());
    
    await act(async () => {
      await result.current.login('test@example.com', 'password123');
    });

    expect(result.current.isAuthenticated).toBe(true);
    expect(result.current.user).toBeDefined();
  });
});
```

#### コンポーネントテスト例

```typescript
// src/components/Button/__tests__/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { Button } from '../Button';

describe('Button', () => {
  it('ラベルが正しく表示される', () => {
    render(<Button label="クリック" onClick={() => {}} />);
    expect(screen.getByText('クリック')).toBeInTheDocument();
  });

  it('クリックイベントが発火する', () => {
    const handleClick = vi.fn();
    render(<Button label="クリック" onClick={handleClick} />);
    
    fireEvent.click(screen.getByText('クリック'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('disabled状態では動作しない', () => {
    const handleClick = vi.fn();
    render(<Button label="クリック" onClick={handleClick} disabled />);
    
    const button = screen.getByText('クリック');
    expect(button).toBeDisabled();
    fireEvent.click(button);
    expect(handleClick).not.toHaveBeenCalled();
  });
});
```

---

### バックエンド: Jest

**対象**:
- サービス層のビジネスロジック
- ユーティリティ関数
- バリデーションロジック

#### サービステスト例

```typescript
// src/modules/attendance/attendance.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { AttendanceService } from './attendance.service';
import { getRepositoryToken } from '@nestjs/typeorm';
import { Attendance } from './entities/attendance.entity';
import { BadRequestException } from '@nestjs/common';
import { Repository } from 'typeorm';

describe('AttendanceService', () => {
  let service: AttendanceService;
  let mockRepository: jest.Mocked<Partial<Repository<Attendance>>>;

  beforeEach(async () => {
    mockRepository = {
      findOne: jest.fn(),
      create: jest.fn(),
      save: jest.fn(),
      find: jest.fn(),
    };

    const module: TestingModule = await Test.createTestingModule({
      providers: [
        AttendanceService,
        {
          provide: getRepositoryToken(Attendance),
          useValue: mockRepository,
        },
      ],
    }).compile();

    service = module.get<AttendanceService>(AttendanceService);
  });

  describe('clockIn', () => {
    it('出勤打刻が正常に記録される', async () => {
      const userId = 'user-123';
      const now = new Date();
      mockRepository.findOne.mockResolvedValue(null);
      mockRepository.create.mockReturnValue({ userId, clockInTime: now } as Partial<Attendance>);
      mockRepository.save.mockResolvedValue({ id: '1', userId, clockInTime: now } as Attendance);

      const result = await service.clockIn(userId);

      expect(result).toBeDefined();
      expect(result.userId).toBe(userId);
      expect(mockRepository.save).toHaveBeenCalled();
    });

    it('既に出勤済みの場合はエラーを投げる', async () => {
      const userId = 'user-123';
      mockRepository.findOne.mockResolvedValue({ userId, clockInTime: new Date() } as Attendance);

      await expect(service.clockIn(userId)).rejects.toThrow(BadRequestException);
    });
  });
});
```

---

## コンポーネントテスト (Storybook)

**目的**:
- コンポーネントの視覚的な確認
- 様々な状態（props）での動作確認
- UIカタログの作成
- デザインシステムの文書化

### Storybook設定

```typescript
// .storybook/main.ts
import type { StorybookConfig } from '@storybook/react-vite';

const config: StorybookConfig = {
  stories: ['../src/**/*.stories.@(js|jsx|ts|tsx)'],
  addons: [
    '@storybook/addon-links',
    '@storybook/addon-essentials',
    '@storybook/addon-interactions',
    '@storybook/addon-a11y',
  ],
  framework: {
    name: '@storybook/react-vite',
    options: {},
  },
  docs: {
    autodocs: 'tag',
  },
};

export default config;
```

### Story例

```typescript
// src/components/Button/Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';

const meta: Meta<typeof Button> = {
  title: 'Components/Button',
  component: Button,
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary'],
    },
    disabled: {
      control: 'boolean',
    },
  },
};

export default meta;
type Story = StoryObj<typeof Button>;

export const Primary: Story = {
  args: {
    label: '出勤打刻',
    variant: 'primary',
    onClick: () => alert('出勤打刻しました'),
  },
};

export const Secondary: Story = {
  args: {
    label: 'キャンセル',
    variant: 'secondary',
    onClick: () => alert('キャンセルしました'),
  },
};

export const Disabled: Story = {
  args: {
    label: '無効ボタン',
    variant: 'primary',
    disabled: true,
    onClick: () => {},
  },
};
```

### Interactionテスト

```typescript
// src/components/LoginForm/LoginForm.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { within, userEvent, expect } from '@storybook/test';
import { LoginForm } from './LoginForm';

const meta: Meta<typeof LoginForm> = {
  title: 'Forms/LoginForm',
  component: LoginForm,
};

export default meta;
type Story = StoryObj<typeof LoginForm>;

export const FilledForm: Story = {
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);

    // メールアドレス入力
    await userEvent.type(
      canvas.getByLabelText('メールアドレス'),
      'user@example.com'
    );

    // パスワード入力
    await userEvent.type(
      canvas.getByLabelText('パスワード'),
      'password123'
    );

    // ボタンが有効になることを確認
    const submitButton = canvas.getByRole('button', { name: 'ログイン' });
    await expect(submitButton).not.toBeDisabled();
  },
};
```

**実行コマンド**:

```bash
# Storybookの起動
npm run storybook

# ビルド
npm run build-storybook
```

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

## テストカバレッジ目標

| テストタイプ | 目標カバレッジ | 対象 |
|------------|-------------|------|
| 単体テスト | 80%以上 | ビジネスロジック、ユーティリティ関数 |
| 統合テスト | 主要APIエンドポイント全て | API層 |
| E2Eテスト | 主要ユーザーフロー全て | システム全体 |

---

## CI/CD統合

### GitHub Actions設定例

```yaml
# .github/workflows/test.yml
name: Test

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  unit-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      # フロントエンド単体テスト
      - name: Frontend Unit Tests
        working-directory: ./frontend
        run: |
          npm ci
          npm run test -- --coverage
      
      # バックエンド単体テスト
      - name: Backend Unit Tests
        working-directory: ./backend
        run: |
          npm ci
          npm run test -- --coverage
      
      - name: Upload Coverage
        uses: codecov/codecov-action@v3

  integration-test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Backend Integration Tests
        working-directory: ./backend
        run: |
          npm ci
          npm run test:e2e
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db

  e2e-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install Playwright
        run: npx playwright install --with-deps
      
      - name: Start Backend
        working-directory: ./backend
        env:
          API_URL: http://localhost:3000/api
          HEALTH_CHECK_TIMEOUT: 60000
        run: |
          npm ci
          npm run start:prod &
          # APIサーバーの起動を待機（環境変数で設定可能、クォートで安全に）
          npx wait-on "${API_URL}" --timeout "${HEALTH_CHECK_TIMEOUT}"
      
      - name: Start Frontend
        working-directory: ./frontend
        env:
          FRONTEND_URL: http://localhost:5173
          HEALTH_CHECK_TIMEOUT: 60000
        run: |
          npm ci
          npm run build
          npm run preview &
          # フロントエンドの起動を待機（環境変数で設定可能、クォートで安全に）
          npx wait-on "${FRONTEND_URL}" --timeout "${HEALTH_CHECK_TIMEOUT}"
      
      - name: Run E2E Tests
        run: |
          cd e2e
          npm ci
          npm run test:e2e
      
      - name: Upload Test Results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: e2e/playwright-report/
```

---

## ベストプラクティス

1. **テストの独立性**: 各テストは他のテストに依存せず独立して実行可能であること
2. **データのセットアップ**: テストデータは各テストで準備し、クリーンアップすること
3. **モックの活用**: 外部依存を適切にモック化すること
4. **明確なテスト名**: テストケース名は何をテストしているか明確にすること
5. **AAA パターン**: Arrange（準備）、Act（実行）、Assert（検証）の順で書くこと
6. **data-testid属性の使用**: E2Eテストでは`data-testid`属性を使用して安定したセレクタを実現すること（国際化対応にも有効）
7. **null安全性**: TypeScriptのnon-null assertion operator (`!`) を避け、適切なnullチェックを行うこと
8. **CI/CD統合**: 全テストがCI/CDパイプラインで自動実行されること
9. **レポート**: テスト結果とカバレッジレポートを可視化すること

---

**関連ドキュメント**:
- [システムアーキテクチャ設計書](./ARCHITECTURE.md)
- [フロントエンド設計](./FRONTEND.md)
- [バックエンド設計](./BACKEND.md)

---

**最終更新日**: 2024年12月17日  
**バージョン**: 1.0.0
