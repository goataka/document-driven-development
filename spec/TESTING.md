# テスト戦略

本ドキュメントは、システム全体のテスト戦略の概要と基本的なテスト（単体テスト・コンポーネントテスト）の実装方法を説明します。
**すべて無料のツールを使用しています。**

**関連ドキュメント**: 
- [システムアーキテクチャ設計書](./ARCHITECTURE.md)
- [統合・E2Eテスト](./TESTING_INTEGRATION.md)
- [品質テスト (スナップショット・アクセシビリティ・パフォーマンス)](./TESTING_QUALITY.md)
- [セキュリティ・脆弱性テスト](./TESTING_SECURITY.md)
- [依存関係管理と自動更新](./TESTING_DEPENDENCIES.md)

## 目次

1. [テストピラミッド](#テストピラミッド)
2. [単体テスト](#単体テスト)
   - [フロントエンド: Vitest + React Testing Library](#フロントエンド-vitest--react-testing-library)
   - [バックエンド: Jest](#バックエンド-jest)
3. [コンポーネントテスト (Storybook)](#コンポーネントテスト-storybook)
4. [テストカバレッジ目標](#テストカバレッジ目標)
5. [CI/CD統合](#cicd統合)
6. [ベストプラクティス](#ベストプラクティス)

**詳細なテスト戦略**:
- **統合テスト・E2Eテスト**: [TESTING_INTEGRATION.md](./TESTING_INTEGRATION.md)を参照
- **スナップショット・アクセシビリティ・パフォーマンステスト**: [TESTING_QUALITY.md](./TESTING_QUALITY.md)を参照
- **セキュリティ・脆弱性テスト**: [TESTING_SECURITY.md](./TESTING_SECURITY.md)を参照
- **依存関係管理**: [TESTING_DEPENDENCIES.md](./TESTING_DEPENDENCIES.md)を参照

---


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

---


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

---

## 関連ドキュメント

- [システムアーキテクチャ設計書](./ARCHITECTURE.md)
- [フロントエンド設計書](./FRONTEND.md)
- [バックエンド設計書](./BACKEND.md)
- [統合・E2Eテスト](./TESTING_INTEGRATION.md)
- [品質テスト](./TESTING_QUALITY.md)
- [セキュリティ・脆弱性テスト](./TESTING_SECURITY.md)
- [依存関係管理と自動更新](./TESTING_DEPENDENCIES.md)
