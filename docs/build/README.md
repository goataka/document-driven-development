# ビルド戦略

本ドキュメントは、システムのビルドプロセスとビルド戦略を説明します。
**すべて無料のツールを使用しています。**

**関連ドキュメント**: 
- [システムアーキテクチャ設計書](../implement/)
- [デプロイ戦略](../deploy/)
- [テスト戦略](../test/)

## 目次

1. [概要](#概要)
2. [ビルド環境](#ビルド環境)
3. [フロントエンドビルド](#フロントエンドビルド)
4. [バックエンドビルド](#バックエンドビルド)
5. [ビルド最適化](#ビルド最適化)
6. [CI/CDビルド](#cicdビルド)
7. [トラブルシューティング](#トラブルシューティング)

---

## 概要

本システムのビルドプロセスは、フロントエンド（React + Vite）とバックエンド（NestJS）の2つの独立したビルドパイプラインで構成されます。

### ビルドの目的

- **コードのトランスパイル**: TypeScriptからJavaScriptへの変換
- **最適化**: コードの圧縮、バンドル化
- **静的アセット生成**: フロントエンドの静的ファイル生成
- **依存関係の解決**: 必要なライブラリのバンドル
- **型チェック**: TypeScriptの型安全性検証

---

## ビルド環境

### 必要な環境

| 項目 | バージョン | 用途 |
|------|-----------|------|
| **Node.js** | 20.x LTS | JavaScriptランタイム |
| **npm** | 10.x | パッケージマネージャー |
| **TypeScript** | 5.x | 型システム |

### 環境変数

#### フロントエンド (.env.production)

```env
VITE_API_URL=https://api.example.com
VITE_APP_NAME=勤怠管理システム
VITE_APP_VERSION=1.0.0
```

#### バックエンド (.env.production)

```env
NODE_ENV=production
PORT=3000
DATABASE_URL=postgresql://user:password@localhost:5432/attendance_db
JWT_SECRET=your-production-secret-key
JWT_EXPIRATION=7d
CORS_ORIGIN=https://example.com
```

---

## フロントエンドビルド

### ビルドプロセス

```bash
# ディレクトリ移動
cd frontend

# 依存関係インストール
npm ci

# 型チェック
npm run type-check

# ビルド実行
npm run build

# ビルド結果は dist/ に出力される
```

### Vite設定

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  build: {
    outDir: 'dist',
    sourcemap: true,
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true, // 本番環境ではconsole.logを削除
      }
    },
    rollupOptions: {
      output: {
        manualChunks: {
          'react-vendor': ['react', 'react-dom', 'react-router-dom'],
          'mui': ['@mui/material', '@mui/icons-material'],
        }
      }
    },
    chunkSizeWarningLimit: 1000,
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    }
  }
});
```

### ビルド成果物

```
frontend/dist/
├── index.html              # エントリーポイント
├── assets/
│   ├── index-[hash].js     # メインJavaScript
│   ├── react-vendor-[hash].js  # Reactライブラリ
│   ├── mui-[hash].js       # Material-UIライブラリ
│   └── index-[hash].css    # スタイルシート
└── favicon.ico
```

---

## バックエンドビルド

### ビルドプロセス

```bash
# ディレクトリ移動
cd backend

# 依存関係インストール
npm ci

# 型チェック
npm run type-check

# ビルド実行
npm run build

# ビルド結果は dist/ に出力される
```

### NestJS設定

```json
// nest-cli.json
{
  "collection": "@nestjs/schematics",
  "sourceRoot": "src",
  "compilerOptions": {
    "deleteOutDir": true,
    "assets": ["**/*.proto"],
    "watchAssets": true
  }
}
```

### TypeScript設定

```json
// tsconfig.build.json
{
  "extends": "./tsconfig.json",
  "exclude": [
    "node_modules",
    "test",
    "dist",
    "**/*spec.ts",
    "**/*test.ts"
  ]
}
```

### ビルド成果物

```
backend/dist/
├── main.js                 # エントリーポイント
├── app.module.js
├── config/
├── common/
├── modules/
│   ├── auth/
│   ├── users/
│   └── attendance/
└── database/
```

---

## ビルド最適化

### フロントエンド最適化

#### 1. コード分割

```typescript
// 動的インポートによるコード分割
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Attendance = lazy(() => import('./pages/Attendance'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/attendance" element={<Attendance />} />
      </Routes>
    </Suspense>
  );
}
```

#### 2. 画像最適化

```typescript
// vite.config.ts
import imagemin from 'vite-plugin-imagemin';

export default defineConfig({
  plugins: [
    imagemin({
      gifsicle: { optimizationLevel: 7 },
      optipng: { optimizationLevel: 7 },
      mozjpeg: { quality: 80 },
      pngquant: { quality: [0.8, 0.9] },
      svgo: {
        plugins: [
          { name: 'removeViewBox', active: false },
          { name: 'removeEmptyAttrs', active: false }
        ]
      }
    })
  ]
});
```

#### 3. キャッシュ戦略

- **ハッシュ化されたファイル名**: 長期キャッシュ可能
- **Vendor分割**: ライブラリの変更頻度を考慮
- **Service Worker**: オフライン対応（オプション）

### バックエンド最適化

#### 1. 依存関係の最適化

```bash
# 本番用依存関係のみインストール
npm ci --production

# 不要なファイルを除外
# .dockerignore, .gitignore を活用
```

#### 2. ビルド時の型チェック

```json
// package.json
{
  "scripts": {
    "prebuild": "npm run type-check",
    "build": "nest build",
    "type-check": "tsc --noEmit"
  }
}
```

---

## CI/CDビルド

### GitHub Actions設定

```yaml
# .github/workflows/build.yml
name: Build

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  build-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json
      
      - name: Install dependencies
        working-directory: ./frontend
        run: npm ci
      
      - name: Type check
        working-directory: ./frontend
        run: npm run type-check
      
      - name: Build
        working-directory: ./frontend
        run: npm run build
      
      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: frontend-dist
          path: frontend/dist/
          retention-days: 7

  build-backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: backend/package-lock.json
      
      - name: Install dependencies
        working-directory: ./backend
        run: npm ci
      
      - name: Type check
        working-directory: ./backend
        run: npm run type-check
      
      - name: Build
        working-directory: ./backend
        run: npm run build
      
      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: backend-dist
          path: backend/dist/
          retention-days: 7
```

### ビルドキャッシュ

```yaml
# キャッシュ戦略
- uses: actions/cache@v3
  with:
    path: |
      ~/.npm
      node_modules
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

---

## トラブルシューティング

### よくある問題と解決方法

#### 1. メモリ不足エラー

```bash
# Node.jsのメモリ上限を増やす
NODE_OPTIONS="--max-old-space-size=4096" npm run build
```

#### 2. 型エラー

```bash
# 型定義を再インストール
rm -rf node_modules package-lock.json
npm install

# 型チェックのみ実行
npm run type-check
```

#### 3. ビルド時間が長い

```typescript
// vite.config.ts - 開発用設定を本番から分離
export default defineConfig(({ mode }) => ({
  build: {
    minify: mode === 'production' ? 'terser' : false,
    sourcemap: mode === 'production' ? true : 'inline',
  }
}));
```

#### 4. 依存関係の競合

```bash
# package-lock.jsonを削除して再インストール
rm package-lock.json
npm install

# または特定のバージョンを指定
npm install package-name@specific-version
```

### ビルドログの確認

```bash
# 詳細ログ付きビルド
npm run build -- --verbose

# ビルドサイズ分析
npm run build -- --report
```

---

## ベストプラクティス

1. **ビルド前のクリーン**: 前回のビルド成果物を削除
2. **型チェックの自動化**: ビルド前に必ず型チェックを実行
3. **環境変数の管理**: `.env.example`を提供し、機密情報は含めない
4. **ビルド成果物の検証**: ビルド後にサイズや構造を確認
5. **キャッシュの活用**: CI/CDでビルドキャッシュを利用
6. **並列ビルド**: フロントエンドとバックエンドを並列実行
7. **アーティファクトの保存**: CI/CDでビルド成果物を保存
8. **バージョン管理**: ビルド時にバージョン情報を埋め込む

---

## 関連ドキュメント

- [システムアーキテクチャ設計書](../implement/)
- [デプロイ戦略](../deploy/)
- [リリース管理](../release/)
- [テスト戦略](../test/)
- [依存関係管理](../implement/DEPENDENCIES.md)

---

**最終更新日**: 2024年12月19日  
**バージョン**: 1.0.0  
**ドキュメント管理者**: 開発チーム
