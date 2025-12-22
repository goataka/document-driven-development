# モノレポ構成 - ワークフロー

このドキュメントは、モノレポでの開発ワークフローとCI/CD設定を説明します。

## 開発ワークフロー

### 1. セットアップ

```bash
# リポジトリのクローン
git clone https://github.com/goataka/spec-driven-development.git
cd spec-driven-development

# 依存関係のインストール
npm install

# 全パッケージのビルド
npm run build

# 開発サーバーの起動（すべてのアプリ）
npm run dev
```

### 2. 機能開発の流れ

```mermaid
graph LR
    A[ブランチ作成] --> B[コード変更]
    B --> C[ローカルテスト]
    C --> D[コミット]
    D --> E[プッシュ]
    E --> F[PR作成]
    F --> G[CI実行]
    G --> H[コードレビュー]
    H --> I[マージ]
    I --> J[自動デプロイ]
```

### 3. ブランチ戦略

#### ブランチの種類

- `main` - 本番環境
- `develop` - 開発環境
- `feature/*` - 機能開発
- `fix/*` - バグ修正
- `docs/*` - ドキュメント更新

#### ブランチ命名規則

```bash
# 機能開発
feature/add-user-profile

# バグ修正
fix/attendance-calculation-bug

# ドキュメント
docs/update-api-docs

# 複数パッケージに影響する変更
multi/update-typescript-version
```

### 4. コミットフロー

```bash
# 変更されたファイルの確認
git status

# ステージング
git add .

# コミット（コミット規約に従う）
git commit -m "feat(frontend): add user profile page"

# プッシュ
git push origin feature/add-user-profile
```

## CI/CD パイプライン

### GitHub Actions 設定

#### 1. PR チェック

```yaml
# .github/workflows/pr-check.yml
name: PR Check

on:
  pull_request:
    branches:
      - main
      - develop

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Lint
        run: npm run lint
      
      - name: Type check
        run: npm run type-check
      
      - name: Test
        run: npm run test
      
      - name: Build
        run: npm run build

  affected:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm ci
      
      # 変更されたパッケージのみテスト
      - name: Test affected
        run: npx turbo run test --filter=...[origin/main]
      
      # 変更されたパッケージのみビルド
      - name: Build affected
        run: npx turbo run build --filter=...[origin/main]
```

#### 2. 自動デプロイ

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches:
      - main

jobs:
  deploy-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build frontend
        run: npm run build --filter=@apps/frontend
      
      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          working-directory: ./apps/frontend

  deploy-backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build backend
        run: npm run build --filter=@apps/backend
      
      - name: Deploy to AWS
        run: |
          # AWSへのデプロイコマンド
          echo "Deploy to AWS"

  deploy-docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.x'
      
      - name: Install MkDocs
        run: pip install mkdocs-material
      
      - name: Build user docs
        run: |
          cd sites/user-docs
          mkdocs build
      
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./sites/user-docs/site
```

#### 3. 依存関係の自動更新

```yaml
# .github/workflows/dependabot-auto-merge.yml
name: Dependabot Auto Merge

on:
  pull_request:
    types:
      - opened
      - synchronize

jobs:
  auto-merge:
    runs-on: ubuntu-latest
    if: github.actor == 'dependabot[bot]'
    steps:
      - name: Dependabot metadata
        id: metadata
        uses: dependabot/fetch-metadata@v1
      
      - name: Auto-merge for patch updates
        if: steps.metadata.outputs.update-type == 'version-update:semver-patch'
        run: gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Turborepo によるタスク実行

### 基本的なタスク

```bash
# すべてのパッケージをビルド
npm run build

# すべてのパッケージをテスト
npm run test

# すべてのパッケージをリント
npm run lint

# すべてのアプリを開発モードで起動
npm run dev
```

### フィルタリング

```bash
# 特定のパッケージのみビルド
npm run build --filter=@apps/frontend

# 特定のパッケージと依存関係をビルド
npm run build --filter=@apps/frontend...

# 複数のパッケージを指定
npm run build --filter=@apps/frontend --filter=@apps/backend

# 変更されたパッケージのみビルド
npm run build --filter=...[HEAD^1]
```

### キャッシュの活用

```bash
# キャッシュを使用してビルド（デフォルト）
npm run build

# キャッシュを無視してビルド
npm run build --force

# リモートキャッシュを使用
npm run build --remote-cache=...
```

## ローカル開発環境

### 複数アプリの同時起動

```json
// package.json
{
  "scripts": {
    "dev": "turbo run dev --parallel",
    "dev:frontend": "turbo run dev --filter=@apps/frontend",
    "dev:backend": "turbo run dev --filter=@apps/backend",
    "dev:docs": "cd sites/user-docs && mkdocs serve"
  }
}
```

### ホットリロード

- フロントエンド: Viteのホットリロード
- バックエンド: Nest.jsのウォッチモード
- 共有パッケージ: TypeScriptのウォッチモード

### デバッグ

#### VS Code 設定

```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Frontend",
      "type": "chrome",
      "request": "launch",
      "url": "http://localhost:3000",
      "webRoot": "${workspaceFolder}/apps/frontend"
    },
    {
      "name": "Debug Backend",
      "type": "node",
      "request": "launch",
      "runtimeExecutable": "npm",
      "runtimeArgs": ["run", "dev:backend"],
      "skipFiles": ["<node_internals>/**"]
    }
  ]
}
```

## テストの実行

### 単体テスト

```bash
# すべてのパッケージのテスト
npm run test

# 特定のパッケージのテスト
npm run test --filter=@repo/utils

# ウォッチモード
npm run test:watch --filter=@apps/frontend

# カバレッジ
npm run test:coverage
```

### E2Eテスト

```bash
# E2Eテストの実行
npm run test:e2e --filter=@apps/frontend

# E2Eテスト（ヘッドレスモード）
npm run test:e2e:headless
```

## リリースワークフロー

### 1. バージョン更新

```bash
# ルートでバージョンアップ
npm version patch  # 1.0.0 -> 1.0.1
npm version minor  # 1.0.0 -> 1.1.0
npm version major  # 1.0.0 -> 2.0.0

# 特定のパッケージのバージョンアップ
cd packages/ui
npm version patch
```

### 2. チェンジログの生成

```bash
# 自動でチェンジログ生成
npm run changelog

# または手動で作成
# docs/release/CHANGELOG.md を更新
```

### 3. リリースタグの作成

```bash
# タグの作成
git tag -a v1.0.0 -m "Release version 1.0.0"

# タグのプッシュ
git push origin v1.0.0
```

### 4. GitHub Release の作成

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Create Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          body_path: ./docs/release/CHANGELOG.md
          draft: false
          prerelease: false
```

## パフォーマンス最適化

### ビルドの高速化

1. **依存関係のキャッシュ**
   ```yaml
   - uses: actions/setup-node@v4
     with:
       cache: 'npm'
   ```

2. **並列実行**
   ```bash
   npm run build --parallel
   ```

3. **増分ビルド**
   - Turborepoのキャッシュを活用
   - 変更されたパッケージのみビルド

### CI/CD の高速化

1. **ジョブの並列化**
   - lint, test, build を並列実行

2. **条件付き実行**
   - ドキュメントのみの変更時はビルドをスキップ

3. **リモートキャッシュ**
   - Vercel Remote Cacheを使用

## トラブルシューティング

### 依存関係の問題

```bash
# node_modules を削除して再インストール
rm -rf node_modules
rm package-lock.json
npm install

# Turborepo のキャッシュをクリア
rm -rf .turbo
```

### ビルドエラー

```bash
# クリーンビルド
npm run clean
npm run build
```

### テストの失敗

```bash
# 特定のテストのみ実行
npm run test -- --testNamePattern="ユーザー登録"

# デバッグモードで実行
npm run test:debug
```

---

**最終更新日**: 2025年12月  
**バージョン**: 1.0.0
