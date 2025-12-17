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

```mermaid
graph TB
    Client[クライアント<br/>Webブラウザ: Chrome, Edge等]
    CDN[CDN / Static Hosting<br/>React SPA - Vite Build]
    Gateway[API Gateway / Load Balancer]
    NestJS[NestJS Application Server]
    Controllers[Controllers<br/>API Endpoints]
    Services[Services<br/>Business Logic]
    Repositories[Repositories<br/>Data Access]
    DB[(PostgreSQL Database<br/>Users, Attendance Records)]
    Redis[(Redis Cache<br/>Session, Token Store<br/>オプション)]
    
    Client -->|HTTPS| CDN
    CDN -->|REST API HTTPS<br/>JSON| Gateway
    Gateway --> NestJS
    NestJS --> Controllers
    Controllers --> Services
    Services --> Repositories
    Repositories --> DB
    NestJS -.->|オプション| Redis
    
    style Client fill:#e1f5ff
    style CDN fill:#fff4e1
    style Gateway fill:#ffe1f5
    style NestJS fill:#f0e1ff
    style Controllers fill:#e1ffe1
    style Services fill:#e1ffe1
    style Repositories fill:#e1ffe1
    style DB fill:#ffe1e1
    style Redis fill:#fff4e1
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

## フロントエンド設計

フロントエンド設計の詳細については、[フロントエンド設計ドキュメント](./FRONTEND.md)を参照してください。

### 概要

- **フレームワーク**: React 18.x + TypeScript
- **ビルドツール**: Vite
- **状態管理**: Zustand（推奨）
- **ルーティング**: React Router
- **UIライブラリ**: Material-UI (MUI)

---

## バックエンド設計

バックエンド設計の詳細については、[バックエンド設計ドキュメント](./BACKEND.md)を参照してください。

### 概要

- **フレームワーク**: NestJS 10.x + TypeScript
- **ORM**: TypeORM（推奨）
- **認証**: Passport.js + JWT
- **API仕様**: Swagger (OpenAPI)

---

## データベース設計

データベース設計の詳細については、[データベース設計ドキュメント](./DATABASE.md)を参照してください。

### 概要

- **RDBMS**: PostgreSQL 15.x
- **主要テーブル**: users, attendances
- **マイグレーション**: TypeORM Migrations

---

## API設計

API設計の詳細については、[API設計ドキュメント](./API.md)を参照してください。

### 概要

- **スタイル**: RESTful API
- **フォーマット**: JSON
- **ドキュメント**: Swagger UI (/api/docs)

---

## セキュリティ

セキュリティ設計の詳細については、[セキュリティドキュメント](./SECURITY.md)を参照してください。

### 概要

- **認証**: JWT トークンベース
- **パスワード**: bcrypt ハッシュ化
- **セキュリティヘッダー**: Helmet
- **レート制限**: Express rate limiter
- **HTTPS**: 本番環境では必須

---

## テスト戦略

テスト戦略の詳細については、[テスト戦略ドキュメント](./TESTING.md)を参照してください。

### 概要

- **単体テスト**: Jest (backend), Vitest (frontend)
- **コンポーネントテスト**: Storybook
- **統合テスト**: Jest + Supertest
- **E2Eテスト**: Cucumber + Playwright

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

**フロントエンド (.env)**:
```env
VITE_API_URL=http://localhost:3000/api
VITE_APP_NAME=勤怠管理システム
```

**バックエンド (.env)**:
```env
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

GitHub Actionsを使用した自動化の詳細は、[テスト戦略ドキュメント](./TESTING.md#cicd統合)を参照してください。

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

#### 技術ドキュメント

- **[フロントエンド設計](./FRONTEND.md)**: React + TypeScriptの詳細設計
- **[バックエンド設計](./BACKEND.md)**: NestJS + TypeScriptの詳細設計
- **[データベース設計](./DATABASE.md)**: PostgreSQLのスキーマ設計
- **[API設計](./API.md)**: RESTful APIの仕様
- **[セキュリティ](./SECURITY.md)**: セキュリティ対策の詳細
- **[テスト戦略](./TESTING.md)**: 包括的なテスト戦略

#### ユーザードキュメント

- **[ユーザーマニュアル](./MANUAL.md)**: エンドユーザー向けマニュアル
- **[リリースノート](./RELEASE_NOTES.md)**: 機能概要とバージョン情報
- **[FAQ](./FAQ.md)**: よくある質問

---

**最終更新日**: 2024年12月17日  
**バージョン**: 3.0.0 (詳細を専門ドキュメントに分離)  
**ドキュメント管理者**: 開発チーム
