# テスト戦略

本ドキュメントは、システム全体のテスト戦略と実装方法の詳細を説明します。

**関連ドキュメント**: [システムアーキテクチャ設計書](./ARCHITECTURE.md)

## 目次

1. [テストピラミッド](#テストピラミッド)
2. [単体テスト](#単体テスト)
3. [コンポーネントテスト (Storybook)](#コンポーネントテスト-storybook)
4. [統合テスト](#統合テスト)
5. [E2Eテスト (Cucumber + Playwright)](#e2eテスト-cucumber--playwright)
6. [スナップショットテスト](#スナップショットテスト)
7. [アクセシビリティテスト](#アクセシビリティテスト)
8. [パフォーマンステスト](#パフォーマンステスト)
9. [セキュリティ・脆弱性テスト](#セキュリティ脆弱性テスト)
10. [依存関係管理と自動更新](#依存関係管理と自動更新)
11. [テストカバレッジ目標](#テストカバレッジ目標)
12. [CI/CD統合](#cicd統合)
13. [ベストプラクティス](#ベストプラクティス)

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

## スナップショットテスト

スナップショットテストは、UIコンポーネントの予期しない変更を検出するために使用します。

### フロントエンド: Vitest + React

#### スナップショットテスト設定

```typescript
// vitest.config.ts（スナップショット設定を追加）
export default defineConfig({
  test: {
    // ... 他の設定
    snapshotSerializers: ['@testing-library/jest-dom/serializers'],
  }
});
```

#### コンポーネントスナップショットテスト例

```typescript
// src/components/Button/__tests__/Button.snapshot.test.tsx
import { render } from '@testing-library/react';
import { describe, it, expect } from 'vitest';
import { Button } from '../Button';

describe('Button スナップショットテスト', () => {
  it('デフォルトのボタンをスナップショット', () => {
    const { container } = render(<Button>クリック</Button>);
    expect(container.firstChild).toMatchSnapshot();
  });

  it('primary variantのボタンをスナップショット', () => {
    const { container } = render(
      <Button variant="primary">送信</Button>
    );
    expect(container.firstChild).toMatchSnapshot();
  });

  it('disabledボタンをスナップショット', () => {
    const { container } = render(
      <Button disabled>無効</Button>
    );
    expect(container.firstChild).toMatchSnapshot();
  });

  it('loadingボタンをスナップショット', () => {
    const { container } = render(
      <Button loading>読み込み中</Button>
    );
    expect(container.firstChild).toMatchSnapshot();
  });
});
```

#### スナップショット更新コマンド

```bash
# スナップショットを更新
npm test -- -u

# 特定のファイルのスナップショットを更新
npm test Button.snapshot.test.tsx -- -u
```

### ベストプラクティス

1. **小さな単位でテスト**: 大きなコンポーネントツリー全体ではなく、小さな単位でスナップショット
2. **頻繁に見直し**: スナップショットの更新時は差分を必ず確認
3. **意図的な変更のみ**: 意図しない変更がないか注意深くレビュー
4. **動的データの除外**: タイムスタンプやランダムIDなど動的データはモック化

---

## アクセシビリティテスト

アクセシビリティテストは、すべてのユーザーがアプリケーションを使用できることを保証します。

### ツール: jest-axe + axe-core

#### インストール

```bash
npm install --save-dev jest-axe axe-core @axe-core/playwright
```

#### jest-axe設定

```typescript
// src/test/setup.ts
import { toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);
```

#### コンポーネントアクセシビリティテスト例

```typescript
// src/components/LoginForm/__tests__/LoginForm.a11y.test.tsx
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';
import { describe, it, expect } from 'vitest';
import { LoginForm } from '../LoginForm';

expect.extend(toHaveNoViolations);

describe('LoginForm アクセシビリティテスト', () => {
  it('WCAG 2.1 Level AA基準に準拠すること', async () => {
    const { container } = render(<LoginForm />);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });

  it('フォーム要素に適切なラベルがあること', async () => {
    const { container, getByLabelText } = render(<LoginForm />);
    
    // ラベルで要素を取得できることを確認
    expect(getByLabelText('メールアドレス')).toBeInTheDocument();
    expect(getByLabelText('パスワード')).toBeInTheDocument();
    
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });

  it('エラーメッセージが適切にアナウンスされること', async () => {
    const { container, getByRole } = render(
      <LoginForm error="認証に失敗しました" />
    );
    
    // aria-liveリージョンが存在することを確認
    const alert = getByRole('alert');
    expect(alert).toHaveTextContent('認証に失敗しました');
    
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });
});
```

### E2E アクセシビリティテスト (Playwright)

```typescript
// e2e/tests/accessibility.spec.ts
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test.describe('アクセシビリティテスト', () => {
  test('トップページがアクセシビリティ基準を満たすこと', async ({ page }) => {
    await page.goto('/');
    
    const accessibilityScanResults = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa'])
      .analyze();
    
    expect(accessibilityScanResults.violations).toEqual([]);
  });

  test('ログインページがキーボード操作可能なこと', async ({ page }) => {
    await page.goto('/login');
    
    // Tab キーで全てのインタラクティブ要素にアクセス可能
    await page.keyboard.press('Tab'); // メールアドレスフィールド
    await expect(page.locator('[name="email"]')).toBeFocused();
    
    await page.keyboard.press('Tab'); // パスワードフィールド
    await expect(page.locator('[name="password"]')).toBeFocused();
    
    await page.keyboard.press('Tab'); // ログインボタン
    await expect(page.locator('button[type="submit"]')).toBeFocused();
    
    const accessibilityScanResults = await new AxeBuilder({ page })
      .analyze();
    
    expect(accessibilityScanResults.violations).toEqual([]);
  });

  test('フォームバリデーションエラーが適切にアナウンスされること', async ({ page }) => {
    await page.goto('/login');
    
    // 空でログインを試行
    await page.click('button[type="submit"]');
    
    // エラーメッセージのaria-live属性を確認
    const errorMessage = page.locator('[role="alert"]');
    await expect(errorMessage).toBeVisible();
    
    const accessibilityScanResults = await new AxeBuilder({ page })
      .analyze();
    
    expect(accessibilityScanResults.violations).toEqual([]);
  });
});
```

### アクセシビリティチェックリスト

- ✅ すべてのフォーム要素に適切な`<label>`がある
- ✅ 画像に代替テキスト（`alt`属性）がある
- ✅ 色だけで情報を伝えていない
- ✅ キーボードだけで全操作が可能
- ✅ フォーカスインジケーターが明確
- ✅ 適切なARIA属性（`role`, `aria-label`, `aria-live`など）
- ✅ 十分なコントラスト比（4.5:1以上）
- ✅ スクリーンリーダーで適切に読み上げられる

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

### パフォーマンス目標

| メトリクス | 目標値 |
|----------|-------|
| First Contentful Paint (FCP) | < 2.0秒 |
| Largest Contentful Paint (LCP) | < 2.5秒 |
| Total Blocking Time (TBT) | < 300ms |
| Cumulative Layout Shift (CLS) | < 0.1 |
| Time to Interactive (TTI) | < 3.5秒 |
| API レスポンス時間 | < 200ms (P95) |

---

## セキュリティ・脆弱性テスト

セキュリティテストは、アプリケーションの脆弱性を検出し、セキュアな実装を保証します。

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

### Snyk

より高度な脆弱性検出とモニタリングにはSnykを使用します。

#### インストールと設定

```bash
# Snyk CLI のインストール
npm install -g snyk

# 認証
snyk auth

# プロジェクトをテスト
snyk test

# 継続的なモニタリング
snyk monitor
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
      
      - name: Run Snyk Security Scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high
      
      - name: Upload Snyk results to GitHub
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: snyk.sarif
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

## 依存関係管理と自動更新

依存関係を最新かつ安全な状態に保つための戦略とツールです。

### Dependabot

GitHub Dependabotを使用して、依存関係の自動更新を設定します。

#### 設定ファイル

```yaml
# .github/dependabot.yml
version: 2
updates:
  # npm dependencies (frontend)
  - package-ecosystem: "npm"
    directory: "/frontend"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
      timezone: "Asia/Tokyo"
    open-pull-requests-limit: 10
    reviewers:
      - "team-developers"
    assignees:
      - "tech-lead"
    labels:
      - "dependencies"
      - "frontend"
    commit-message:
      prefix: "chore(deps):"
    # セキュリティアップデートは即座にマージ
    # 通常のアップデートはレビュー後にマージ
    versioning-strategy: increase
    
  # npm dependencies (backend)
  - package-ecosystem: "npm"
    directory: "/backend"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
      timezone: "Asia/Tokyo"
    open-pull-requests-limit: 10
    reviewers:
      - "team-developers"
    assignees:
      - "tech-lead"
    labels:
      - "dependencies"
      - "backend"
    commit-message:
      prefix: "chore(deps):"
    versioning-strategy: increase
    
  # GitHub Actions
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
      timezone: "Asia/Tokyo"
    labels:
      - "dependencies"
      - "ci-cd"
    commit-message:
      prefix: "chore(ci):"
```

### Renovate（代替オプション）

Renovate は Dependabot よりも柔軟な設定が可能です。

```json
// renovate.json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:base"],
  "schedule": ["every weekend"],
  "timezone": "Asia/Tokyo",
  "labels": ["dependencies"],
  "packageRules": [
    {
      "matchUpdateTypes": ["minor", "patch"],
      "matchCurrentVersion": "!/^0/",
      "automerge": true,
      "automergeType": "pr",
      "automergeStrategy": "squash"
    },
    {
      "matchDepTypes": ["devDependencies"],
      "matchUpdateTypes": ["minor", "patch"],
      "automerge": true
    },
    {
      "matchPackagePatterns": ["^@types/"],
      "automerge": true
    },
    {
      "matchPackageNames": ["typescript", "eslint", "prettier"],
      "groupName": "linting and formatting"
    },
    {
      "matchPackagePatterns": ["^@testing-library/", "^vitest", "^jest"],
      "groupName": "testing tools"
    }
  ],
  "vulnerabilityAlerts": {
    "labels": ["security"],
    "assignees": ["@team-security"]
  }
}
```

### 依存関係チェックのワークフロー

```yaml
# .github/workflows/dependency-check.yml
name: Dependency Check

on:
  schedule:
    # 毎週月曜日午前9時に実行
    - cron: '0 0 * * 1'
  workflow_dispatch:

jobs:
  check-dependencies:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Check for outdated packages
        run: npm outdated || true
      
      - name: Check for security vulnerabilities
        run: npm audit --audit-level=moderate
      
      - name: Run Snyk test
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      
      - name: Check for deprecated packages
        run: npx check-dependencies
      
      - name: Generate dependency report
        run: |
          echo "# 依存関係レポート" > dependency-report.md
          echo "## Outdated Packages" >> dependency-report.md
          npm outdated --json >> dependency-report.md || true
          echo "## Security Audit" >> dependency-report.md
          npm audit --json >> dependency-report.md || true
      
      - name: Upload dependency report
        uses: actions/upload-artifact@v4
        with:
          name: dependency-report
          path: dependency-report.md
```

### npm-check-updates

手動で依存関係を更新する場合に便利なツールです。

```bash
# npm-check-updates のインストール
npm install -g npm-check-updates

# 更新可能なパッケージを確認
ncu

# 全てのパッケージを最新版に更新
ncu -u

# インストール
npm install

# テスト実行
npm test
```

### 依存関係管理のベストプラクティス

1. **定期的な更新**: 週次でDependabotによる自動更新を実行
2. **セキュリティパッチ優先**: セキュリティアップデートは即座に適用
3. **グループ化**: 関連する依存関係をグループ化して一括更新
4. **自動マージ**: パッチ・マイナーバージョンの更新は自動マージ
5. **メジャーバージョン**: 手動レビュー必須
6. **テスト必須**: CI/CDでの自動テストをパス後にマージ
7. **ロックファイルのコミット**: `package-lock.json` を必ずコミット
8. **監査ログ**: 更新履歴とテスト結果を記録

### package.json のバージョン管理戦略

```json
{
  "dependencies": {
    // ピン留め（推奨：本番環境の重要なライブラリ）
    "react": "18.2.0",
    
    // マイナー・パッチ更新許可（推奨：安定したライブラリ）
    "axios": "^1.6.0",
    
    // パッチ更新のみ許可（慎重な場合）
    "lodash": "~4.17.21"
  },
  "devDependencies": {
    // 開発ツールは柔軟に更新可能
    "typescript": "^5.3.0",
    "vitest": "^1.0.0"
  }
}
```

---

## テストカバレッジ目標

| テストタイプ | 目標カバレッジ | 対象 |
|------------|-------------|------|
| 単体テスト | 80%以上 | ビジネスロジック、ユーティリティ関数 |
| スナップショットテスト | 全UIコンポーネント | UIコンポーネントの視覚的回帰 |
| 統合テスト | 主要APIエンドポイント全て | API層 |
| E2Eテスト | 主要ユーザーフロー全て | システム全体 |
| アクセシビリティテスト | 全ページ・コンポーネント | WCAG 2.1 Level AA準拠 |
| パフォーマンステスト | 全主要ページ | Core Web Vitals目標達成 |
| セキュリティテスト | 全APIエンドポイント | 脆弱性ゼロ |

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
10. **スナップショットレビュー**: スナップショット更新時は差分を必ず確認すること
11. **アクセシビリティ優先**: 全UIコンポーネントでアクセシビリティテストを実施すること
12. **パフォーマンス監視**: 定期的にパフォーマンステストを実行し、劣化を早期検出すること
13. **セキュリティファースト**: セキュリティテストを開発サイクルに組み込むこと
14. **依存関係の最新化**: 定期的に依存関係を更新し、脆弱性を解消すること
15. **テスト自動化**: 手動テストを最小限にし、自動化可能なテストは全て自動化すること

---

**関連ドキュメント**:
- [システムアーキテクチャ設計書](./ARCHITECTURE.md)
- [フロントエンド設計](./FRONTEND.md)
- [バックエンド設計](./BACKEND.md)

---

**最終更新日**: 2024年12月18日  
**バージョン**: 2.0.0
