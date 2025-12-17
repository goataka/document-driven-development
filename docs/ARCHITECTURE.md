# システムアーキテクチャ設計書

## 目次

1. [概要](#概要)
2. [設計原則](#設計原則)
3. [技術スタック](#技術スタック)
4. [システムアーキテクチャ](#システムアーキテクチャ)
5. [フロントエンド設計](#フロントエンド設計)
6. [バックエンド設計](#バックエンド設計)
7. [データベース設計](#データベース設計)
8. [API設計](#api設計)
9. [セキュリティ](#セキュリティ)
10. [開発環境とデプロイ](#開発環境とデプロイ)

---

## 概要

本ドキュメントは、勤怠管理システムのアーキテクチャ設計方針を定義します。システムは**React**（フロントエンド）と**NestJS**（バックエンド）をベースとしたモダンなウェブアプリケーションとして構築されます。

### 基本方針

- **SSR（Server-Side Rendering）は不要**: クライアントサイドレンダリング（CSR）のSPA（Single Page Application）として構築
- **型安全性**: TypeScriptを全面的に採用し、型安全な開発を実現
- **モジュール性**: 機能ごとに疎結合なモジュール構造を採用
- **保守性**: コードの可読性と保守性を重視した設計
- **スケーラビリティ**: 将来的な機能拡張を考慮した拡張可能な設計

---

## 設計原則

### 1. レイヤードアーキテクチャ

システムは以下の3層に分離されます：

- **プレゼンテーション層**（Frontend）: ユーザーインターフェース
- **アプリケーション層**（Backend API）: ビジネスロジック
- **データ層**（Database）: データ永続化

### 2. 関心の分離（Separation of Concerns）

- フロントエンドとバックエンドを完全に分離
- APIを介した通信によりフロントエンドとバックエンドを疎結合に保つ
- 各層内でも責務を明確に分離（例: コンポーネント、サービス、リポジトリ）

### 3. 再利用性

- 共通コンポーネントの積極的な活用
- ビジネスロジックの共通化
- ユーティリティ関数の集約

### 4. テスタビリティ

- 単体テスト可能な設計
- 依存性注入（DI）の活用
- モックやスタブの容易な実装

---

## 技術スタック

### フロントエンド

| カテゴリ | 技術 | バージョン | 用途 |
|---------|------|-----------|------|
| **フレームワーク** | React | 18.x | UIライブラリ |
| **言語** | TypeScript | 5.x | 型安全な開発 |
| **ビルドツール** | Vite | 5.x | 高速な開発サーバーとビルド |
| **状態管理** | Zustand / Redux Toolkit | 最新 | グローバル状態管理 |
| **ルーティング** | React Router | 6.x | クライアントサイドルーティング |
| **UIライブラリ** | Material-UI (MUI) / Ant Design | 5.x | UIコンポーネント |
| **フォーム管理** | React Hook Form | 7.x | フォームバリデーション |
| **HTTP通信** | Axios | 1.x | APIリクエスト |
| **日付処理** | date-fns | 3.x | 日付の操作と表示 |
| **テスト** | Vitest + React Testing Library | 最新 | 単体・統合テスト |
| **リンター/フォーマッター** | ESLint + Prettier | 最新 | コード品質管理 |

### バックエンド

| カテゴリ | 技術 | バージョン | 用途 |
|---------|------|-----------|------|
| **フレームワーク** | NestJS | 10.x | サーバーサイドフレームワーク |
| **言語** | TypeScript | 5.x | 型安全な開発 |
| **ランタイム** | Node.js | 20.x LTS | JavaScriptランタイム |
| **ORM** | TypeORM / Prisma | 最新 | データベースアクセス |
| **バリデーション** | class-validator + class-transformer | 最新 | リクエストバリデーション |
| **認証** | Passport.js + JWT | 最新 | 認証・認可 |
| **API仕様** | Swagger (OpenAPI) | 最新 | API仕様書自動生成 |
| **テスト** | Jest | 最新 | 単体・統合テスト |
| **リンター/フォーマッター** | ESLint + Prettier | 最新 | コード品質管理 |

### データベース

| カテゴリ | 技術 | バージョン | 用途 |
|---------|------|-----------|------|
| **RDBMS** | PostgreSQL | 15.x | メインデータベース |
| **キャッシュ** | Redis | 7.x | セッション管理、キャッシュ（オプション） |

### インフラ・DevOps

| カテゴリ | 技術 | 用途 |
|---------|------|------|
| **コンテナ** | Docker + Docker Compose | ローカル開発環境 |
| **CI/CD** | GitHub Actions | 自動テスト・デプロイ |
| **ホスティング** | AWS / GCP / Azure / Vercel + Heroku | 本番環境（要検討） |
| **監視** | Sentry / CloudWatch | エラー監視・ログ管理 |

---

## システムアーキテクチャ

### 全体構成図

```
┌─────────────────────────────────────────────────────────────┐
│                         クライアント                          │
│                    (Webブラウザ: Chrome, Edge等)              │
└────────────────────────────┬────────────────────────────────┘
                             │ HTTPS
                             │
┌────────────────────────────▼────────────────────────────────┐
│                     CDN / Static Hosting                     │
│                   (React SPA - Vite Build)                   │
└────────────────────────────┬────────────────────────────────┘
                             │ REST API (HTTPS)
                             │ JSON
┌────────────────────────────▼────────────────────────────────┐
│                     API Gateway / Load Balancer              │
└────────────────────────────┬────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────┐
│                    NestJS Application Server                 │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Controllers (API Endpoints)                         │   │
│  └─────────────────┬────────────────────────────────────┘   │
│  ┌─────────────────▼────────────────────────────────────┐   │
│  │  Services (Business Logic)                           │   │
│  └─────────────────┬────────────────────────────────────┘   │
│  ┌─────────────────▼────────────────────────────────────┐   │
│  │  Repositories (Data Access)                          │   │
│  └─────────────────┬────────────────────────────────────┘   │
└────────────────────┼────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│                     PostgreSQL Database                      │
│              (Users, Attendance Records, etc.)               │
└──────────────────────────────────────────────────────────────┘

オプション:
┌──────────────────────────────────────────────────────────────┐
│                         Redis Cache                          │
│                  (Session, Token Store)                      │
└──────────────────────────────────────────────────────────────┘
```

### データフロー

1. **クライアント → バックエンド**
   - ユーザーがReact UIで操作
   - AxiosでHTTP/HTTPSリクエスト送信
   - JWTトークンをAuthorizationヘッダーに付与

2. **バックエンド処理**
   - NestJSのGuardで認証・認可チェック
   - Controllerがリクエストを受信
   - Serviceでビジネスロジック実行
   - Repositoryでデータベースアクセス

3. **バックエンド → クライアント**
   - JSON形式でレスポンス返却
   - エラー時は適切なHTTPステータスコードとメッセージ

---

## フロントエンド設計

### ディレクトリ構成

```
frontend/
├── public/                    # 静的ファイル
│   └── favicon.ico
├── src/
│   ├── main.tsx              # エントリーポイント
│   ├── App.tsx               # ルートコンポーネント
│   ├── routes/               # ルーティング設定
│   │   └── index.tsx
│   ├── pages/                # ページコンポーネント
│   │   ├── Login/
│   │   ├── Register/
│   │   ├── Dashboard/
│   │   ├── ClockInOut/
│   │   └── AttendanceHistory/
│   ├── components/           # 共通コンポーネント
│   │   ├── common/           # 汎用コンポーネント
│   │   │   ├── Button/
│   │   │   ├── Input/
│   │   │   └── Modal/
│   │   └── layouts/          # レイアウトコンポーネント
│   │       ├── Header/
│   │       ├── Sidebar/
│   │       └── Footer/
│   ├── hooks/                # カスタムフック
│   │   ├── useAuth.ts
│   │   └── useAttendance.ts
│   ├── store/                # 状態管理
│   │   ├── authStore.ts
│   │   └── attendanceStore.ts
│   ├── services/             # API通信
│   │   ├── api.ts            # Axios設定
│   │   ├── authService.ts
│   │   └── attendanceService.ts
│   ├── types/                # TypeScript型定義
│   │   ├── user.ts
│   │   └── attendance.ts
│   ├── utils/                # ユーティリティ関数
│   │   ├── dateFormatter.ts
│   │   └── validator.ts
│   ├── constants/            # 定数
│   │   └── index.ts
│   └── styles/               # グローバルスタイル
│       └── global.css
├── package.json
├── tsconfig.json
├── vite.config.ts
└── .env.example
```

### 主要設計パターン

#### 1. コンポーネント設計

- **Atomic Design**の考え方を部分的に採用
- **関数コンポーネント + React Hooks**を使用
- **プレゼンテーションコンポーネント**と**コンテナコンポーネント**を分離

```typescript
// 例: Buttonコンポーネント
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

#### 2. 状態管理

- **ローカル状態**: useState, useReducer
- **グローバル状態**: Zustand または Redux Toolkit
- **サーバー状態**: React Query（TanStack Query）の導入を推奨

```typescript
// 例: Zustandストア
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

#### 3. ルーティング

```typescript
// 例: ルート定義
import { createBrowserRouter } from 'react-router-dom';

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

#### 4. API通信

```typescript
// 例: Axios設定
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

## バックエンド設計

### ディレクトリ構成

```
backend/
├── src/
│   ├── main.ts                      # エントリーポイント
│   ├── app.module.ts                # ルートモジュール
│   ├── config/                      # 設定ファイル
│   │   ├── database.config.ts
│   │   └── jwt.config.ts
│   ├── common/                      # 共通モジュール
│   │   ├── filters/                 # 例外フィルター
│   │   ├── guards/                  # 認証ガード
│   │   ├── interceptors/            # インターセプター
│   │   ├── decorators/              # カスタムデコレーター
│   │   └── pipes/                   # バリデーションパイプ
│   ├── modules/                     # 機能モジュール
│   │   ├── auth/                    # 認証モジュール
│   │   │   ├── auth.module.ts
│   │   │   ├── auth.controller.ts
│   │   │   ├── auth.service.ts
│   │   │   ├── dto/
│   │   │   │   ├── login.dto.ts
│   │   │   │   └── register.dto.ts
│   │   │   └── strategies/
│   │   │       └── jwt.strategy.ts
│   │   ├── users/                   # ユーザーモジュール
│   │   │   ├── users.module.ts
│   │   │   ├── users.controller.ts
│   │   │   ├── users.service.ts
│   │   │   ├── entities/
│   │   │   │   └── user.entity.ts
│   │   │   └── dto/
│   │   │       └── create-user.dto.ts
│   │   └── attendance/              # 勤怠モジュール
│   │       ├── attendance.module.ts
│   │       ├── attendance.controller.ts
│   │       ├── attendance.service.ts
│   │       ├── entities/
│   │       │   └── attendance.entity.ts
│   │       └── dto/
│   │           ├── clock-in.dto.ts
│   │           ├── clock-out.dto.ts
│   │           └── get-history.dto.ts
│   └── database/                    # データベース関連
│       ├── migrations/              # マイグレーション
│       └── seeds/                   # シードデータ
├── test/                            # テスト
│   ├── unit/
│   └── e2e/
├── package.json
├── tsconfig.json
├── nest-cli.json
└── .env.example
```

### 主要設計パターン

#### 1. モジュール構成

NestJSのモジュールシステムを活用し、機能ごとに独立したモジュールを作成：

```typescript
// 例: AuthModule
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

#### 2. コントローラー設計

```typescript
// 例: AttendanceController
@Controller('attendance')
@UseGuards(JwtAuthGuard)
@ApiBearerAuth()
@ApiTags('attendance')
export class AttendanceController {
  constructor(private readonly attendanceService: AttendanceService) {}

  @Post('clock-in')
  @ApiOperation({ summary: '出勤打刻' })
  @ApiResponse({ status: 201, description: '打刻成功' })
  async clockIn(@GetUser() user: User): Promise<AttendanceRecord> {
    return this.attendanceService.clockIn(user.id);
  }

  @Post('clock-out')
  @ApiOperation({ summary: '退勤打刻' })
  async clockOut(@GetUser() user: User): Promise<AttendanceRecord> {
    return this.attendanceService.clockOut(user.id);
  }

  @Get('history')
  @ApiOperation({ summary: '勤怠履歴取得' })
  async getHistory(
    @GetUser() user: User,
    @Query() query: GetHistoryDto
  ): Promise<AttendanceRecord[]> {
    return this.attendanceService.getHistory(user.id, query);
  }
}
```

#### 3. サービス層

```typescript
// 例: AttendanceService
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

#### 4. DTO（Data Transfer Object）

```typescript
// 例: Clock-in DTO
export class ClockInDto {
  @ApiProperty({ description: '打刻位置情報（オプション）' })
  @IsOptional()
  @IsObject()
  location?: {
    latitude: number;
    longitude: number;
  };
}

// 例: Get History DTO
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

#### 5. エンティティ定義

```typescript
// 例: User Entity
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

  @Column({ type: 'timestamp', default: () => 'CURRENT_TIMESTAMP' })
  createdAt: Date;

  @Column({ type: 'timestamp', default: () => 'CURRENT_TIMESTAMP' })
  @UpdateDateColumn()
  updatedAt: Date;

  @OneToMany(() => Attendance, (attendance) => attendance.user)
  attendances: Attendance[];
}

// 例: Attendance Entity
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

  @Column({ type: 'timestamp', default: () => 'CURRENT_TIMESTAMP' })
  createdAt: Date;

  @Column({ type: 'timestamp', default: () => 'CURRENT_TIMESTAMP' })
  @UpdateDateColumn()
  updatedAt: Date;
}
```

---

## データベース設計

### テーブル設計

#### 1. users テーブル

| カラム名 | 型 | 制約 | 説明 |
|---------|-----|-----|-----|
| id | UUID | PRIMARY KEY | ユーザーID |
| email | VARCHAR(255) | UNIQUE, NOT NULL | メールアドレス |
| password | VARCHAR(255) | NOT NULL | ハッシュ化パスワード |
| company_code | VARCHAR(50) | NOT NULL | 会社コード |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 作成日時 |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 更新日時 |

#### 2. attendances テーブル

| カラム名 | 型 | 制約 | 説明 |
|---------|-----|-----|-----|
| id | UUID | PRIMARY KEY | 勤怠記録ID |
| user_id | UUID | FOREIGN KEY (users.id), NOT NULL | ユーザーID |
| clock_in_time | TIMESTAMP | NOT NULL | 出勤時刻 |
| clock_out_time | TIMESTAMP | NULL | 退勤時刻 |
| work_duration_minutes | INTEGER | NULL | 勤務時間（分） |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 作成日時 |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 更新日時 |

### インデックス設計

```sql
-- users テーブル
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_company_code ON users(company_code);

-- attendances テーブル
CREATE INDEX idx_attendances_user_id ON attendances(user_id);
CREATE INDEX idx_attendances_clock_in_time ON attendances(clock_in_time);
CREATE INDEX idx_attendances_user_clock_in ON attendances(user_id, clock_in_time);
```

### マイグレーション戦略

- **TypeORM Migrations**または**Prisma Migrate**を使用
- 本番環境へのマイグレーションは自動化せず、手動実行を推奨
- ロールバック計画を事前に準備
- バックアップを必ず取得してから実行

---

## API設計

### RESTful API原則

- **リソース指向**: URL設計はリソースを表現
- **HTTPメソッド**: GET, POST, PUT/PATCH, DELETEを適切に使用
- **ステータスコード**: 適切なHTTPステータスコードを返却
- **JSON形式**: リクエスト・レスポンスはJSON

### エンドポイント一覧

#### 認証API

| メソッド | エンドポイント | 説明 | 認証 |
|---------|--------------|------|-----|
| POST | /api/auth/register | ユーザー登録 | 不要 |
| POST | /api/auth/login | ログイン | 不要 |
| POST | /api/auth/logout | ログアウト | 必要 |
| GET | /api/auth/me | 現在のユーザー情報取得 | 必要 |

#### ユーザーAPI

| メソッド | エンドポイント | 説明 | 認証 |
|---------|--------------|------|-----|
| GET | /api/users/profile | プロフィール取得 | 必要 |
| PUT | /api/users/profile | プロフィール更新 | 必要 |

#### 勤怠API

| メソッド | エンドポイント | 説明 | 認証 |
|---------|--------------|------|-----|
| POST | /api/attendance/clock-in | 出勤打刻 | 必要 |
| POST | /api/attendance/clock-out | 退勤打刻 | 必要 |
| GET | /api/attendance/history | 勤怠履歴取得 | 必要 |
| GET | /api/attendance/today | 本日の打刻状況 | 必要 |

### レスポンス形式

#### 成功時

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "clockInTime": "2024-01-01T09:00:00Z",
    "clockOutTime": null
  }
}
```

#### エラー時

```json
{
  "success": false,
  "error": {
    "code": "ALREADY_CLOCKED_IN",
    "message": "既に出勤打刻済みです",
    "details": {}
  }
}
```

### APIドキュメント

- **Swagger UI**を使用してAPIドキュメントを自動生成
- エンドポイント: `/api/docs`
- 全APIエンドポイントの仕様、リクエスト/レスポンス例を記載

---

## セキュリティ

### 認証・認可

#### 1. JWT（JSON Web Token）認証

- ログイン時にJWTトークンを発行
- トークンの有効期限: 7日間
- リフレッシュトークンの実装を推奨（将来的な拡張）

#### 2. パスワード管理

- **bcrypt**を使用したハッシュ化（salt rounds: 10以上）
- パスワードポリシー:
  - 最小8文字
  - 英大文字・小文字・数字を含む
  - 特殊文字を推奨

#### 3. 認可制御

- ユーザーは自分自身のデータのみアクセス可能
- 会社コードによる組織分離

### セキュリティ対策

#### 1. CORS設定

```typescript
// main.ts
app.enableCors({
  origin: process.env.FRONTEND_URL,
  credentials: true
});
```

#### 2. ヘルメット（Helmet）

```typescript
import helmet from 'helmet';
app.use(helmet());
```

#### 3. レート制限

```typescript
import rateLimit from 'express-rate-limit';

app.use(
  rateLimit({
    windowMs: 15 * 60 * 1000, // 15分
    max: 100 // 最大100リクエスト
  })
);
```

#### 4. バリデーション

- すべてのユーザー入力をバリデーション
- **class-validator**を使用したDTOレベルのバリデーション
- SQLインジェクション対策（ORMの使用）
- XSS対策（入力のサニタイゼーション）

#### 5. HTTPS強制

- 本番環境では必ずHTTPSを使用
- HTTP Strict Transport Security (HSTS)ヘッダーの設定

---

## 開発環境とデプロイ

### ローカル開発環境

#### 前提条件

- Node.js 20.x LTS
- npm または yarn
- Docker & Docker Compose
- PostgreSQL 15.x（Dockerで実行可）

#### セットアップ手順

```bash
# リポジトリクローン
git clone <repository-url>
cd document-driven-development

# フロントエンド
cd frontend
npm install
cp .env.example .env
npm run dev

# バックエンド
cd backend
npm install
cp .env.example .env
npm run start:dev

# データベース（Docker Compose）
docker-compose up -d postgres
```

### 環境変数

#### フロントエンド (.env)

```
VITE_API_URL=http://localhost:3000/api
VITE_APP_NAME=勤怠管理システム
```

#### バックエンド (.env)

```
NODE_ENV=development
PORT=3000
DATABASE_URL=postgresql://user:password@localhost:5432/attendance_db
JWT_SECRET=your-secret-key-change-in-production
JWT_EXPIRATION=7d
FRONTEND_URL=http://localhost:5173
```

### ビルドとデプロイ

#### フロントエンド

```bash
# ビルド
npm run build

# プレビュー
npm run preview

# デプロイ（例: Vercel）
vercel --prod
```

#### バックエンド

```bash
# ビルド
npm run build

# 本番起動
npm run start:prod

# マイグレーション実行
npm run migration:run
```

### CI/CDパイプライン

GitHub Actionsを使用した自動化：

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'
      - name: Install dependencies
        run: npm ci
      - name: Run linter
        run: npm run lint
      - name: Run tests
        run: npm test
      - name: Build
        run: npm run build
```

### デプロイ先候補

#### フロントエンド

- **Vercel**: 推奨、SPAに最適
- **Netlify**: 代替案
- **AWS S3 + CloudFront**: 大規模向け

#### バックエンド

- **Heroku**: 簡単なデプロイ
- **AWS ECS/Fargate**: コンテナベース
- **GCP Cloud Run**: サーバーレスコンテナ
- **Azure App Service**: Microsoftエコシステム

#### データベース

- **AWS RDS**: マネージドPostgreSQL
- **Heroku Postgres**: 統合管理
- **Supabase**: BaaS（Backend as a Service）

---

## まとめ

本アーキテクチャ設計書は、React + NestJSをベースとした勤怠管理システムの実装方針を定義しました。

### 重要なポイント

1. **フロントエンドとバックエンドの完全分離**: APIを介した疎結合な設計
2. **TypeScriptによる型安全性**: 開発効率とコード品質の向上
3. **モジュラーアーキテクチャ**: 機能ごとの独立性と保守性
4. **セキュリティファースト**: 認証・認可、バリデーション、HTTPS
5. **スケーラビリティ**: 将来的な機能拡張を考慮した設計

### 次のステップ

1. 詳細設計（画面設計、API仕様詳細、DB詳細設計）
2. 開発環境のセットアップ
3. 実装フェーズ（スプリント計画）
4. テスト計画の策定
5. デプロイ戦略の確定

---

**最終更新日**: 2024年12月17日  
**バージョン**: 1.0.0  
**ドキュメント管理者**: 開発チーム
