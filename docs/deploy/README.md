# デプロイ戦略

本ドキュメントは、システムのデプロイメント戦略とインフラストラクチャ構成を説明します。

**関連ドキュメント**: 
- [システムアーキテクチャ設計書](../implement/)
- [ビルド戦略](../build/)
- [リリース管理](../release/)
- [運用管理](../operate/)

## 目次

1. [概要](#概要)
2. [デプロイメント環境](#デプロイメント環境)
3. [フロントエンドデプロイ](#フロントエンドデプロイ)
4. [バックエンドデプロイ](#バックエンドデプロイ)
5. [データベース管理](#データベース管理)
6. [デプロイメント戦略](#デプロイメント戦略)
7. [CI/CD パイプライン](#cicd-パイプライン)
8. [環境変数管理](#環境変数管理)

---

## 概要

本システムのデプロイメントは、フロントエンド（静的ホスティング）とバックエンド（コンテナベース）の2つの独立したデプロイメントパイプラインで構成されます。

### デプロイメントの原則

- **自動化**: CI/CDによる自動デプロイ
- **再現性**: 環境の一貫性を保証
- **安全性**: 段階的デプロイとロールバック機能
- **スケーラビリティ**: 負荷に応じた自動スケーリング
- **監視**: デプロイ後の継続的な監視

---

## デプロイメント環境

### 環境構成

| 環境 | 用途 | URL例 | 自動デプロイ |
|------|------|-------|------------|
| **開発 (Development)** | 開発者のローカル環境 | localhost:5173 | - |
| **ステージング (Staging)** | 本番前のテスト環境 | staging.example.com | ✓ |
| **本番 (Production)** | 実運用環境 | example.com | ✓ (タグのみ) |

### インフラストラクチャ構成

```mermaid
graph TB
    Users[ユーザー]
    CDN[CDN / Vercel]
    LB[ロードバランサー]
    API1[API Server 1]
    API2[API Server 2]
    DB[(PostgreSQL<br/>Primary)]
    DBReplica[(PostgreSQL<br/>Replica)]
    Redis[(Redis Cache)]
    
    Users --> CDN
    Users --> LB
    LB --> API1
    LB --> API2
    API1 --> DB
    API2 --> DB
    DB --> DBReplica
    API1 --> Redis
    API2 --> Redis
    
    style Users fill:#e1f5ff
    style CDN fill:#fff4e1
    style LB fill:#ffe1f5
    style API1 fill:#e1ffe1
    style API2 fill:#e1ffe1
    style DB fill:#ffe1e1
    style DBReplica fill:#ffe1e1
    style Redis fill:#fff4e1
```

---

## フロントエンドデプロイ

### Vercel デプロイ（推奨）

#### 初回セットアップ

```bash
# Vercel CLIをインストール
npm install -g vercel

# ログイン
vercel login

# プロジェクトをリンク
cd frontend
vercel link
```

#### デプロイコマンド

```bash
# プレビューデプロイ
vercel

# 本番デプロイ
vercel --prod
```

#### vercel.json 設定

```json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "framework": "vite",
  "env": {
    "VITE_API_URL": "https://api.example.com"
  },
  "routes": [
    {
      "src": "/assets/(.*)",
      "headers": {
        "cache-control": "public, max-age=31536000, immutable"
      }
    },
    {
      "src": "/(.*)",
      "dest": "/index.html"
    }
  ]
}
```

### Netlify デプロイ（代替案）

#### netlify.toml 設定

```toml
[build]
  command = "npm run build"
  publish = "dist"

[build.environment]
  NODE_VERSION = "20"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

[[headers]]
  for = "/assets/*"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"
```

### AWS S3 + CloudFront デプロイ

```bash
# ビルド
npm run build

# S3にアップロード
aws s3 sync dist/ s3://your-bucket-name/ --delete

# CloudFrontキャッシュを無効化
aws cloudfront create-invalidation \
  --distribution-id YOUR_DISTRIBUTION_ID \
  --paths "/*"
```

---

## バックエンドデプロイ

### Docker コンテナ化

#### Dockerfile

```dockerfile
# Multi-stage build
FROM node:20-alpine AS builder

WORKDIR /app

# 依存関係のインストール
COPY package*.json ./
RUN npm ci --only=production

# アプリケーションのコピーとビルド
COPY . .
RUN npm run build

# Production image
FROM node:20-alpine

WORKDIR /app

# 必要なファイルのみコピー
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./

# ヘルスチェック
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# 非rootユーザーで実行
USER node

EXPOSE 3000

CMD ["node", "dist/main.js"]
```

#### docker-compose.yml（ローカル開発用）

```yaml
version: '3.8'

services:
  backend:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:password@db:5432/attendance_db
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=attendance_db
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    restart: unless-stopped

volumes:
  postgres_data:
```

### Heroku デプロイ

#### Procfile

```
web: node dist/main.js
release: npm run migration:run
```

#### heroku.yml（コンテナデプロイ）

```yaml
build:
  docker:
    web: Dockerfile
run:
  web: node dist/main.js
```

#### デプロイコマンド

```bash
# Heroku CLI でログイン
heroku login

# アプリケーション作成
heroku create your-app-name

# PostgreSQL アドオン追加
heroku addons:create heroku-postgresql:mini

# Redis アドオン追加（オプション）
heroku addons:create heroku-redis:mini

# 環境変数設定
heroku config:set NODE_ENV=production
heroku config:set JWT_SECRET=your-secret-key

# デプロイ
git push heroku main
```

### AWS ECS/Fargate デプロイ

#### タスク定義例

```json
{
  "family": "attendance-backend",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "containerDefinitions": [
    {
      "name": "backend",
      "image": "your-ecr-repo/attendance-backend:latest",
      "portMappings": [
        {
          "containerPort": 3000,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "NODE_ENV",
          "value": "production"
        }
      ],
      "secrets": [
        {
          "name": "DATABASE_URL",
          "valueFrom": "arn:aws:secretsmanager:region:account:secret:db-url"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/attendance-backend",
          "awslogs-region": "ap-northeast-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

---

## データベース管理

### マイグレーション実行

#### 開発環境

```bash
# マイグレーション作成
npm run migration:create -- -n CreateUsersTable

# マイグレーション実行
npm run migration:run

# マイグレーション取り消し
npm run migration:revert
```

#### 本番環境

```bash
# マイグレーションを安全に実行
# 1. バックアップ取得
pg_dump -h localhost -U user attendance_db > backup_$(date +%Y%m%d_%H%M%S).sql

# 2. マイグレーション実行
npm run migration:run -- --dry-run  # ドライラン
npm run migration:run               # 実行

# 3. 確認
npm run migration:show
```

### データベースバックアップ

```bash
# 自動バックアップスクリプト
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups"
DB_NAME="attendance_db"

# バックアップ実行
pg_dump -h $DB_HOST -U $DB_USER $DB_NAME | gzip > "$BACKUP_DIR/backup_$DATE.sql.gz"

# 古いバックアップを削除（7日以上前）
find $BACKUP_DIR -name "backup_*.sql.gz" -mtime +7 -delete
```

---

## デプロイメント戦略

### 1. ローリングデプロイ（標準）

```mermaid
graph LR
    A[v1.0 Instance 1] --> B[v1.1 Instance 1]
    C[v1.0 Instance 2] --> D[v1.1 Instance 2]
    E[v1.0 Instance 3] --> F[v1.1 Instance 3]
    
    style B fill:#90EE90
    style D fill:#90EE90
    style F fill:#90EE90
```

- **利点**: ダウンタイムなし、段階的な更新
- **欠点**: 一時的に異なるバージョンが混在

### 2. ブルーグリーンデプロイ

```mermaid
graph TB
    LB[ロードバランサー]
    Blue[Blue 環境<br/>v1.0]
    Green[Green 環境<br/>v1.1]
    
    LB -->|トラフィック切り替え| Green
    LB -.->|旧環境| Blue
    
    style Green fill:#90EE90
    style Blue fill:#D3D3D3
```

- **利点**: 即座に切り替え可能、簡単なロールバック
- **欠点**: リソースが2倍必要

### 3. カナリアリリース

```mermaid
graph TB
    Users[100% ユーザー]
    LB[ロードバランサー]
    V1[v1.0<br/>90%]
    V2[v1.1<br/>10%]
    
    Users --> LB
    LB -->|90%| V1
    LB -->|10%| V2
    
    style V2 fill:#90EE90
```

- **利点**: リスクを最小化、段階的な展開
- **欠点**: 複雑な設定、監視が必要

---

## CI/CD パイプライン

### GitHub Actions デプロイ設定

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]
    tags:
      - 'v*'

jobs:
  deploy-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Build Frontend
        working-directory: ./frontend
        run: |
          npm ci
          npm run build
        env:
          VITE_API_URL: ${{ secrets.PRODUCTION_API_URL }}
      
      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
          working-directory: ./frontend

  deploy-backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-1
      
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2
      
      - name: Build and push Docker image
        working-directory: ./backend
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: attendance-backend
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          docker tag $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPOSITORY:latest
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest
      
      - name: Deploy to ECS
        run: |
          aws ecs update-service \
            --cluster attendance-cluster \
            --service attendance-backend \
            --force-new-deployment
```

---

## 環境変数管理

### 環境変数の管理方法

#### 開発環境

```bash
# .env.development
NODE_ENV=development
DATABASE_URL=postgresql://user:password@localhost:5432/attendance_db
JWT_SECRET=dev-secret-key
```

#### 本番環境

- **AWS Secrets Manager**: 機密情報の管理
- **GitHub Secrets**: CI/CD用の環境変数
- **Heroku Config Vars**: Heroku環境変数

### 環境変数の暗号化

```bash
# AWS Secrets Manager に保存
aws secretsmanager create-secret \
  --name prod/attendance/database \
  --secret-string '{"username":"user","password":"secure-password"}'

# GitHub Secrets に保存
# リポジトリ Settings > Secrets and variables > Actions
```

---

## ベストプラクティス

1. **自動化**: すべてのデプロイを自動化
2. **バージョン管理**: デプロイするコードを明確にタグ付け
3. **環境の分離**: 開発、ステージング、本番を明確に分離
4. **ロールバック計画**: 常にロールバック手順を準備
5. **データベースマイグレーション**: 本番前に十分にテスト
6. **セキュリティ**: 環境変数や機密情報を安全に管理
7. **監視**: デプロイ後の監視を強化
8. **ドキュメント化**: デプロイ手順を常に最新に保つ
9. **テスト**: デプロイ前に包括的なテストを実施
10. **通知**: デプロイの成否を関係者に通知

---

## 関連ドキュメント

- [システムアーキテクチャ設計書](../implement/)
- [ビルド戦略](../build/)
- [リリース管理](../release/)
- [運用管理](../operate/)
- [監視戦略](../monitor/)

---

**最終更新日**: 2024年12月19日  
**バージョン**: 1.0.0  
**ドキュメント管理者**: 開発チーム
