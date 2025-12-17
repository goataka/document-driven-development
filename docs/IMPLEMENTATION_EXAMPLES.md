# 実装例集

本ドキュメントは、[システムアーキテクチャ設計書](./ARCHITECTURE.md)で定義された設計方針に基づく具体的な実装例を提供します。

## 目次

1. [フロントエンド実装例](#フロントエンド実装例)
2. [バックエンド実装例](#バックエンド実装例)
3. [テスト実装例](#テスト実装例)
4. [CI/CD設定例](#cicd設定例)

---

## フロントエンド実装例

### コンポーネント実装

#### Buttonコンポーネント

```typescript
// src/components/common/Button/Button.tsx
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
}

export const Button: React.FC<ButtonProps> = ({
  label,
  onClick,
  variant = 'primary',
  disabled = false
}) => {
  return (
    <button
      className={`btn btn-${variant}`}
      onClick={onClick}
      disabled={disabled}
    >
      {label}
    </button>
  );
};
```

### 状態管理実装

#### Zustandストア例

```typescript
// src/store/authStore.ts
import { create } from 'zustand';

interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

export const useAuthStore = create<AuthState>((set) => ({
  user: null,
  token: null,
  isAuthenticated: false,
  login: async (email, password) => {
    const { user, token } = await authService.login(email, password);
    set({ user, token, isAuthenticated: true });
  },
  logout: () => {
    set({ user: null, token: null, isAuthenticated: false });
  }
}));
```

### ルーティング実装

```typescript
// src/routes/index.tsx
import { createBrowserRouter, Navigate } from 'react-router-dom';
import { Layout } from '../components/layouts/Layout';
import { Login } from '../pages/Login';
import { Register } from '../pages/Register';
import { Dashboard } from '../pages/Dashboard';
import { ClockInOut } from '../pages/ClockInOut';
import { AttendanceHistory } from '../pages/AttendanceHistory';
import { ProtectedRoute } from '../components/ProtectedRoute';

export const router = createBrowserRouter([
  {
    path: '/',
    element: <Layout />,
    children: [
      { index: true, element: <Navigate to="/dashboard" replace /> },
      { path: 'login', element: <Login /> },
      { path: 'register', element: <Register /> },
      {
        path: 'dashboard',
        element: <ProtectedRoute><Dashboard /></ProtectedRoute>
      },
      {
        path: 'clock',
        element: <ProtectedRoute><ClockInOut /></ProtectedRoute>
      },
      {
        path: 'history',
        element: <ProtectedRoute><AttendanceHistory /></ProtectedRoute>
      }
    ]
  }
]);
```

### API通信実装

#### Axios設定

```typescript
// src/services/api.ts
import axios from 'axios';

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL || 'http://localhost:3000/api',
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json'
  }
});

// リクエストインターセプター（トークン付与）
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// レスポンスインターセプター（エラーハンドリング）
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // トークン無効 -> ログアウト処理
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default api;
```

---

## バックエンド実装例

### モジュール構成

#### AuthModule

```typescript
// src/modules/auth/auth.module.ts
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { PassportModule } from '@nestjs/passport';
import { UsersModule } from '../users/users.module';
import { AuthController } from './auth.controller';
import { AuthService } from './auth.service';
import { JwtStrategy } from './strategies/jwt.strategy';

@Module({
  imports: [
    UsersModule,
    PassportModule,
    JwtModule.register({
      secret: process.env.JWT_SECRET,
      signOptions: { expiresIn: '7d' }
    })
  ],
  controllers: [AuthController],
  providers: [AuthService, JwtStrategy],
  exports: [AuthService]
})
export class AuthModule {}
```

### コントローラー実装

#### AttendanceController

```typescript
// src/modules/attendance/attendance.controller.ts
import { Controller, Post, Get, Query, UseGuards } from '@nestjs/common';
import { ApiTags, ApiOperation, ApiResponse, ApiBearerAuth } from '@nestjs/swagger';
import { JwtAuthGuard } from '../auth/guards/jwt-auth.guard';
import { GetUser } from '../auth/decorators/get-user.decorator';
import { AttendanceService } from './attendance.service';
import { User } from '../users/entities/user.entity';
import { GetHistoryDto } from './dto/get-history.dto';
import { Attendance } from './entities/attendance.entity';

@Controller('attendance')
@UseGuards(JwtAuthGuard)
@ApiBearerAuth()
@ApiTags('attendance')
export class AttendanceController {
  constructor(private readonly attendanceService: AttendanceService) {}

  @Post('clock-in')
  @ApiOperation({ summary: '出勤打刻' })
  @ApiResponse({ status: 201, description: '打刻成功' })
  async clockIn(@GetUser() user: User): Promise<Attendance> {
    return this.attendanceService.clockIn(user.id);
  }

  @Post('clock-out')
  @ApiOperation({ summary: '退勤打刻' })
  async clockOut(@GetUser() user: User): Promise<Attendance> {
    return this.attendanceService.clockOut(user.id);
  }

  @Get('history')
  @ApiOperation({ summary: '勤怠履歴取得' })
  async getHistory(
    @GetUser() user: User,
    @Query() query: GetHistoryDto
  ): Promise<Attendance[]> {
    return this.attendanceService.getHistory(user.id, query);
  }
}
```

### サービス層実装

#### AttendanceService

```typescript
// src/modules/attendance/attendance.service.ts
import { Injectable, BadRequestException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository, Not, IsNull, Between } from 'typeorm';
import { Attendance } from './entities/attendance.entity';
import { GetHistoryDto } from './dto/get-history.dto';

@Injectable()
export class AttendanceService {
  constructor(
    @InjectRepository(Attendance)
    private attendanceRepository: Repository<Attendance>
  ) {}

  async clockIn(userId: string): Promise<Attendance> {
    // 既に出勤打刻済みかチェック
    const existing = await this.attendanceRepository.findOne({
      where: {
        userId,
        clockInTime: Not(IsNull()),
        clockOutTime: IsNull()
      }
    });

    if (existing) {
      throw new BadRequestException('既に出勤打刻済みです');
    }

    const attendance = this.attendanceRepository.create({
      userId,
      clockInTime: new Date()
    });

    return this.attendanceRepository.save(attendance);
  }

  async clockOut(userId: string): Promise<Attendance> {
    const attendance = await this.attendanceRepository.findOne({
      where: {
        userId,
        clockInTime: Not(IsNull()),
        clockOutTime: IsNull()
      }
    });

    if (!attendance) {
      throw new BadRequestException('出勤打刻が見つかりません');
    }

    attendance.clockOutTime = new Date();
    return this.attendanceRepository.save(attendance);
  }

  async getHistory(
    userId: string,
    query: GetHistoryDto
  ): Promise<Attendance[]> {
    const { startDate, endDate } = query;
    
    return this.attendanceRepository.find({
      where: {
        userId,
        clockInTime: Between(startDate, endDate)
      },
      order: { clockInTime: 'DESC' }
    });
  }
}
```

### DTO実装

#### GetHistoryDto

```typescript
// src/modules/attendance/dto/get-history.dto.ts
import { ApiProperty } from '@nestjs/swagger';
import { IsDateString, IsOptional, IsInt, Min, Max } from 'class-validator';

export class GetHistoryDto {
  @ApiProperty({ description: '開始日', example: '2024-01-01' })
  @IsDateString()
  startDate: string;

  @ApiProperty({ description: '終了日', example: '2024-01-31' })
  @IsDateString()
  endDate: string;

  @ApiProperty({ description: 'ページ番号', example: 1, required: false })
  @IsOptional()
  @IsInt()
  @Min(1)
  page?: number = 1;

  @ApiProperty({ description: '1ページあたりの件数', example: 20, required: false })
  @IsOptional()
  @IsInt()
  @Min(1)
  @Max(100)
  limit?: number = 20;
}
```

### エンティティ定義

#### User Entity

```typescript
// src/modules/users/entities/user.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, OneToMany, Index, CreateDateColumn, UpdateDateColumn } from 'typeorm';
import { Attendance } from '../../attendance/entities/attendance.entity';

@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  @Index()
  email: string;

  @Column()
  password: string;

  @Column()
  companyCode: string;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @OneToMany(() => Attendance, (attendance) => attendance.user)
  attendances: Attendance[];
}
```

#### Attendance Entity

```typescript
// src/modules/attendance/entities/attendance.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne, JoinColumn, Index, CreateDateColumn, UpdateDateColumn } from 'typeorm';
import { User } from '../../users/entities/user.entity';

@Entity('attendances')
export class Attendance {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  @Index()
  userId: string;

  @ManyToOne(() => User, (user) => user.attendances)
  @JoinColumn({ name: 'userId' })
  user: User;

  @Column({ type: 'timestamp' })
  @Index()
  clockInTime: Date;

  @Column({ type: 'timestamp', nullable: true })
  clockOutTime: Date | null;

  @Column({ type: 'int', nullable: true })
  workDurationMinutes: number | null;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

### セキュリティ実装

#### CORS設定

```typescript
// src/main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.enableCors({
    origin: process.env.FRONTEND_URL,
    credentials: true
  });
  
  await app.listen(3000);
}
bootstrap();
```

#### Helmet実装

```typescript
import helmet from 'helmet';
app.use(helmet());
```

#### レート制限

```typescript
import rateLimit from 'express-rate-limit';

app.use(
  rateLimit({
    windowMs: 15 * 60 * 1000, // 15分
    max: 100 // 最大100リクエスト
  })
);
```

---

## テスト実装例

### 単体テスト

#### フロントエンド: Vitest設定

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

#### フロントエンド: フックテスト例

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

#### フロントエンド: コンポーネントテスト例

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

#### バックエンド: サービステスト例

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

### コンポーネントテスト: Storybook

#### Storybook設定

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

#### Story例

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

#### Interactionテスト

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

### 統合テスト: Jest + Supertest

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

### E2Eテスト: Cucumber + Playwright

#### Feature例

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

#### ステップ定義

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
  // セキュリティ: 特殊文字をより包括的にエスケープ
  const escapedText = buttonText.replace(/['"\\<>]/g, (char) => {
    const escapes: Record<string, string> = {
      "'": "\\'", '"': '\\"', '\\': '\\\\', '<': '&lt;', '>': '&gt;'
    };
    return escapes[char] || char;
  });
  // data-testid属性を優先（より安全）、フォールバックとしてテキスト検索
  const testIdSelector = `[data-testid="${escapedText}-button"]`;
  const textSelector = `button:has-text("${escapedText}")`;
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

#### Page Object例

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

#### Playwright設定

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

---

## CI/CD設定例

### GitHub Actions設定

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

### 環境変数設定例

#### フロントエンド (.env)

```env
VITE_API_URL=http://localhost:3000/api
VITE_APP_NAME=勤怠管理システム
```

#### バックエンド (.env)

```env
NODE_ENV=development
PORT=3000
DATABASE_URL=postgresql://user:password@localhost:5432/attendance_db
JWT_SECRET=your-secret-key-change-in-production
JWT_EXPIRATION=7d
FRONTEND_URL=http://localhost:5173
```

---

**最終更新日**: 2024年12月17日  
**バージョン**: 1.0.0  
**関連ドキュメント**: [システムアーキテクチャ設計書](./ARCHITECTURE.md)
