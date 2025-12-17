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
10. [テスト戦略](#テスト戦略)
11. [開発環境とデプロイ](#開発環境とデプロイ)

---

## 概要

本ドキュメントは、勤怠管理システムのアーキテクチャ設計方針を定義します。システムは**React**（フロントエンド）と**NestJS**（バックエンド）をベースとしたモダンなウェブアプリケーションとして構築されます。

**実装例**: 具体的なコード例については、[実装例集](./IMPLEMENTATION_EXAMPLES.md)を参照してください。

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
| **状態管理** | Zustand（推奨） | 最新 | グローバル状態管理（シンプル・軽量）<br/>※複雑な状態管理が必要な場合はRedux Toolkitも検討可 |
| **ルーティング** | React Router | 6.x | クライアントサイドルーティング |
| **UIライブラリ** | Material-UI (MUI) | 5.x | UIコンポーネント（推奨）<br/>※デザイン要件により他ライブラリも検討可 |
| **フォーム管理** | React Hook Form | 7.x | フォームバリデーション |
| **HTTP通信** | Axios | 1.x | APIリクエスト |
| **日付処理** | date-fns | 3.x | 日付の操作と表示 |
| **単体テスト** | Vitest + React Testing Library | 最新 | コンポーネント・フック単体テスト |
| **コンポーネントテスト** | Storybook | 最新 | UIコンポーネントの視覚的テスト・カタログ化 |
| **E2Eテスト** | Cucumber + Playwright | 最新 | エンドツーエンドテスト（BDD） |
| **リンター/フォーマッター** | ESLint + Prettier | 最新 | コード品質管理 |

### バックエンド

| カテゴリ | 技術 | バージョン | 用途 |
|---------|------|-----------|------|
| **フレームワーク** | NestJS | 10.x | サーバーサイドフレームワーク |
| **言語** | TypeScript | 5.x | 型安全な開発 |
| **ランタイム** | Node.js | 20.x LTS | JavaScriptランタイム |
| **ORM** | TypeORM（推奨） | 最新 | データベースアクセス<br/>※NestJSとの統合が良好、デコレータベース<br/>※型安全性重視の場合はPrismaも検討可 |
| **バリデーション** | class-validator + class-transformer | 最新 | リクエストバリデーション |
| **認証** | Passport.js + JWT | 最新 | 認証・認可 |
| **API仕様** | Swagger (OpenAPI) | 最新 | API仕様書自動生成 |
| **単体テスト** | Jest | 最新 | サービス・コントローラー単体テスト |
| **統合テスト** | Jest + Supertest | 最新 | APIエンドポイント統合テスト |
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

**実装例**: [Buttonコンポーネント実装](./IMPLEMENTATION_EXAMPLES.md#buttonコンポーネント)を参照

#### 2. 状態管理

- **ローカル状態**: useState, useReducer
- **グローバル状態**: Zustand または Redux Toolkit
- **サーバー状態**: React Query（TanStack Query）の導入を推奨

**実装例**: [Zustandストア実装](./IMPLEMENTATION_EXAMPLES.md#zustandストア例)を参照

#### 3. ルーティング

- React Routerを使用したクライアントサイドルーティング
- ProtectedRouteコンポーネントで認証制御
- ネストされたルート構造

**実装例**: [ルーティング実装](./IMPLEMENTATION_EXAMPLES.md#ルーティング実装)を参照

#### 4. API通信

- Axiosインスタンスの作成と設定
- インターセプターによる共通処理（トークン付与、エラーハンドリング）
- 型安全なAPI呼び出し

**実装例**: [Axios設定実装](./IMPLEMENTATION_EXAMPLES.md#axios設定)を参照

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
│   │   │   └── strategies/
│   │   ├── users/                   # ユーザーモジュール
│   │   │   ├── users.module.ts
│   │   │   ├── users.controller.ts
│   │   │   ├── users.service.ts
│   │   │   ├── entities/
│   │   │   └── dto/
│   │   └── attendance/              # 勤怠モジュール
│   │       ├── attendance.module.ts
│   │       ├── attendance.controller.ts
│   │       ├── attendance.service.ts
│   │       ├── entities/
│   │       └── dto/
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
- 各モジュールは必要な機能のみをインポート
- プロバイダーの適切なエクスポート設定
- 疎結合な設計

**実装例**: [AuthModule実装](./IMPLEMENTATION_EXAMPLES.md#authmodule)を参照

#### 2. コントローラー設計

- RESTful APIの原則に従ったエンドポイント設計
- Swaggerデコレーターによる自動ドキュメント生成
- ガードによる認証・認可制御
- DTOによるバリデーション

**実装例**: [AttendanceController実装](./IMPLEMENTATION_EXAMPLES.md#attendancecontroller)を参照

#### 3. サービス層

- ビジネスロジックの実装場所
- データベース操作のカプセル化
- トランザクション管理
- エラーハンドリング

**実装例**: [AttendanceService実装](./IMPLEMENTATION_EXAMPLES.md#attendanceservice)を参照

#### 4. DTO（Data Transfer Object）

- class-validatorによるバリデーション
- class-transformerによる型変換
- Swaggerデコレーターによるドキュメント化

**実装例**: [GetHistoryDto実装](./IMPLEMENTATION_EXAMPLES.md#gethistorydto)を参照

#### 5. エンティティ定義

- TypeORMデコレーターを使用
- リレーションの定義
- インデックスの設定
- 自動タイムスタンプ

**実装例**: [User Entity実装](./IMPLEMENTATION_EXAMPLES.md#user-entity)、[Attendance Entity実装](./IMPLEMENTATION_EXAMPLES.md#attendance-entity)を参照

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

環境変数で許可するオリジンを設定し、本番環境では厳格に管理

**実装例**: [CORS設定実装](./IMPLEMENTATION_EXAMPLES.md#cors設定)を参照

#### 2. ヘルメット（Helmet）

セキュリティヘッダーの設定によるXSS、クリックジャッキング等の対策

**実装例**: [Helmet実装](./IMPLEMENTATION_EXAMPLES.md#helmet実装)を参照

#### 3. レート制限

DDoS攻撃やブルートフォース攻撃の防止

**実装例**: [レート制限実装](./IMPLEMENTATION_EXAMPLES.md#レート制限)を参照

#### 4. バリデーション

- すべてのユーザー入力をバリデーション
- **class-validator**を使用したDTOレベルのバリデーション
- SQLインジェクション対策（ORMの使用）
- XSS対策（入力のサニタイゼーション）

#### 5. HTTPS強制

- 本番環境では必ずHTTPSを使用
- HTTP Strict Transport Security (HSTS)ヘッダーの設定

---

## テスト戦略

テストは品質保証の要であり、以下の複数のレイヤーで包括的にテストを実施します。

### テストピラミッド

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

### 1. 単体テスト（Unit Tests）

#### フロントエンド: Vitest + React Testing Library

**対象**:
- ユーティリティ関数
- カスタムフック
- 状態管理ロジック
- 個別のReactコンポーネント

**実装例**: 
- [Vitest設定](./IMPLEMENTATION_EXAMPLES.md#フロントエンド-vitest設定)
- [フックテスト](./IMPLEMENTATION_EXAMPLES.md#フロントエンド-フックテスト例)
- [コンポーネントテスト](./IMPLEMENTATION_EXAMPLES.md#フロントエンド-コンポーネントテスト例)

#### バックエンド: Jest

**対象**:
- サービス層のビジネスロジック
- ユーティリティ関数
- バリデーションロジック

**実装例**: [サービステスト](./IMPLEMENTATION_EXAMPLES.md#バックエンド-サービステスト例)を参照

### 2. コンポーネントテスト: Storybook

**目的**:
- コンポーネントの視覚的な確認
- 様々な状態（props）での動作確認
- UIカタログの作成
- デザインシステムの文書化

**実装例**: 
- [Storybook設定](./IMPLEMENTATION_EXAMPLES.md#storybook設定)
- [Story例](./IMPLEMENTATION_EXAMPLES.md#story例)
- [Interactionテスト](./IMPLEMENTATION_EXAMPLES.md#interactionテスト)

### 3. 統合テスト（Integration Tests）

#### バックエンド統合テスト: Jest + Supertest

**対象**:
- APIエンドポイントの動作確認
- 認証・認可フローの確認
- データベースとの連携確認

**実装例**: [統合テスト例](./IMPLEMENTATION_EXAMPLES.md#統合テスト-jest--supertest)を参照

### 4. E2Eテスト: Cucumber + Playwright

**目的**:
- ユーザーシナリオ全体の動作確認
- ブラウザでの実際の挙動確認
- ビジネス要件の検証

**技術スタック**:
- **Cucumber**: BDD（振る舞い駆動開発）フレームワーク、Gherkin記法
- **Playwright**: クロスブラウザ自動化ツール

**実装例**: 
- [Feature例](./IMPLEMENTATION_EXAMPLES.md#feature例)
- [ステップ定義](./IMPLEMENTATION_EXAMPLES.md#ステップ定義)
- [Page Object](./IMPLEMENTATION_EXAMPLES.md#page-object例)
- [Playwright設定](./IMPLEMENTATION_EXAMPLES.md#playwright設定)

### テストカバレッジ目標

| テストタイプ | 目標カバレッジ | 対象 |
|------------|-------------|------|
| 単体テスト | 80%以上 | ビジネスロジック、ユーティリティ関数 |
| 統合テスト | 主要APIエンドポイント全て | API層 |
| E2Eテスト | 主要ユーザーフロー全て | システム全体 |

### ベストプラクティス

1. **テストの独立性**: 各テストは他のテストに依存せず独立して実行可能であること
2. **データのセットアップ**: テストデータは各テストで準備し、クリーンアップすること
3. **モックの活用**: 外部依存を適切にモック化すること
4. **明確なテスト名**: テストケース名は何をテストしているか明確にすること
5. **AAA パターン**: Arrange（準備）、Act（実行）、Assert（検証）の順で書くこと
6. **data-testid属性の使用**: E2Eテストでは`data-testid`属性を使用して安定したセレクタを実現すること（国際化対応にも有効）
7. **null安全性**: TypeScriptのnon-null assertion operator (`!`) を避け、適切なnullチェックを行うこと
8. **CI/CD統合**: 全テストがCI/CDパイプラインで自動実行されること
9. **レポート**: テスト結果とカバレッジレポートを可視化すること

**実装例**: [CI/CD設定例](./IMPLEMENTATION_EXAMPLES.md#cicd設定例)を参照

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

**実装例**: [環境変数設定例](./IMPLEMENTATION_EXAMPLES.md#環境変数設定例)を参照

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

GitHub Actionsを使用した自動化

**実装例**: [GitHub Actions設定](./IMPLEMENTATION_EXAMPLES.md#github-actions設定)を参照

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
4. **包括的なテスト戦略**: 単体テスト（Jest/Vitest）、コンポーネントテスト（Storybook）、E2Eテスト（Cucumber + Playwright）
5. **セキュリティファースト**: 認証・認可、バリデーション、HTTPS
6. **スケーラビリティ**: 将来的な機能拡張を考慮した設計

### 次のステップ

1. 詳細設計（画面設計、API仕様詳細、DB詳細設計）
2. 開発環境のセットアップ
3. 実装フェーズ（スプリント計画）
4. テスト計画の策定
5. デプロイ戦略の確定

### 関連ドキュメント

- **[実装例集](./IMPLEMENTATION_EXAMPLES.md)**: 具体的なコード実装例
- **[ユーザーマニュアル](./MANUAL.md)**: エンドユーザー向けマニュアル
- **[リリースノート](./RELEASE_NOTES.md)**: 機能概要とバージョン情報

---

**最終更新日**: 2024年12月17日  
**バージョン**: 2.0.0 (実装例を分離)  
**ドキュメント管理者**: 開発チーム
