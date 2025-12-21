# モノレポ構成 - 依存関係管理

このドキュメントは、モノレポ内のパッケージ間の依存関係管理方法を説明します。

## 依存関係の種類

### 1. 内部パッケージの依存関係

モノレポ内のパッケージ間の依存関係。

```json
// apps/frontend/package.json
{
  "name": "@apps/frontend",
  "dependencies": {
    "@repo/ui": "workspace:*",
    "@repo/types": "workspace:*",
    "@repo/utils": "workspace:*"
  }
}
```

### 2. 外部パッケージの依存関係

npmレジストリからインストールする外部ライブラリ。

```json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  }
}
```

### 3. 開発依存関係

開発時のみ必要なパッケージ。

```json
{
  "devDependencies": {
    "typescript": "^5.0.0",
    "@types/react": "^18.0.0",
    "vite": "^4.0.0"
  }
}
```

## パッケージマネージャー

### npm workspaces（推奨）

```json
// ルート package.json
{
  "name": "document-driven-development",
  "private": true,
  "workspaces": [
    "apps/*",
    "packages/*",
    "sites/*"
  ]
}
```

**メリット**:
- npmに組み込まれている
- シンプルな設定
- 広く使用されている

### pnpm（代替案）

```yaml
# pnpm-workspace.yaml
packages:
  - 'apps/*'
  - 'packages/*'
  - 'sites/*'
```

**メリット**:
- ディスク容量の節約
- 高速なインストール
- 厳格な依存関係管理

### Yarn Workspaces（代替案）

```json
// package.json
{
  "private": true,
  "workspaces": {
    "packages": [
      "apps/*",
      "packages/*"
    ]
  }
}
```

## 依存関係の追加

### 内部パッケージの追加

```bash
# apps/frontend から @repo/ui を使用
cd apps/frontend
npm install @repo/ui@workspace:*

# または、ルートから
npm install @repo/ui@workspace:* --workspace=@apps/frontend
```

### 外部パッケージの追加

```bash
# 特定のアプリに追加
npm install react --workspace=@apps/frontend

# すべてのワークスペースに追加（非推奨）
npm install lodash --workspaces

# ルートに追加（共通の開発依存関係）
npm install -D typescript
```

## 依存関係の管理戦略

### 1. バージョンの統一

同じ外部パッケージは同じバージョンを使用：

```json
// ルート package.json で管理
{
  "devDependencies": {
    "typescript": "^5.0.0",
    "eslint": "^8.0.0",
    "prettier": "^3.0.0"
  }
}
```

### 2. 依存関係の配置

#### ルートに配置するもの
- 開発ツール（TypeScript, ESLint, Prettier）
- ビルドツール（Turborepo, Vite）
- テストフレームワーク（Jest, Vitest）

#### 各パッケージに配置するもの
- ランタイム依存関係
- パッケージ固有の依存関係

### 3. ピア依存関係

共有パッケージで使用する場合：

```json
// packages/ui/package.json
{
  "name": "@repo/ui",
  "peerDependencies": {
    "react": "^18.0.0",
    "react-dom": "^18.0.0"
  },
  "devDependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  }
}
```

## 依存関係グラフ

### 視覚化

```bash
# npm で依存関係を表示
npm list --workspaces

# 特定のパッケージの依存関係
npm list --workspace=@apps/frontend

# Turborepo でグラフを表示
npx turbo run build --graph
```

### 依存関係の例

```
@apps/frontend
├── @repo/ui
│   ├── @repo/types
│   └── react
├── @repo/types
├── @repo/utils
│   └── @repo/types
└── @repo/config

@apps/backend
├── @repo/types
├── @repo/utils
│   └── @repo/types
└── @repo/config
```

## 循環依存の防止

### 検出方法

```bash
# madge を使用
npm install -g madge
madge --circular --extensions ts,tsx ./apps
```

### 回避策

1. **共通の型定義を別パッケージに**
   ```
   @repo/types に共通の型を配置
   ```

2. **依存の方向を統一**
   ```
   下位レイヤー → 上位レイヤー への依存のみ許可
   ```

3. **インターフェースの分離**
   ```typescript
   // 循環依存を避けるためにインターフェースを分離
   export interface IUserService {
     getUser(id: string): Promise<User>;
   }
   ```

## バージョン管理

### セマンティックバージョニング

```
MAJOR.MINOR.PATCH

例: 1.2.3
- MAJOR: 破壊的変更
- MINOR: 後方互換性のある機能追加
- PATCH: 後方互換性のあるバグ修正
```

### 内部パッケージのバージョニング

#### オプション1: 統一バージョン

すべてのパッケージを同じバージョンに保つ：

```json
{
  "name": "@repo/ui",
  "version": "1.0.0"
}

{
  "name": "@repo/utils",
  "version": "1.0.0"
}
```

#### オプション2: 独立バージョン

各パッケージで独立してバージョン管理：

```json
{
  "name": "@repo/ui",
  "version": "2.5.0"
}

{
  "name": "@repo/utils",
  "version": "1.2.0"
}
```

### バージョン更新のタイミング

1. **破壊的変更時**: MAJOR バージョンアップ
2. **新機能追加時**: MINOR バージョンアップ
3. **バグ修正時**: PATCH バージョンアップ

## 依存関係の更新

### 手動更新

```bash
# 特定のパッケージを更新
npm update react --workspace=@apps/frontend

# すべての依存関係を更新
npm update --workspaces
```

### Dependabot による自動更新

```yaml
# .github/dependabot.yml
version: 2
updates:
  # ルートの依存関係
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    groups:
      development-dependencies:
        dependency-type: "development"
        update-types:
          - "minor"
          - "patch"

  # フロントエンドの依存関係
  - package-ecosystem: "npm"
    directory: "/apps/frontend"
    schedule:
      interval: "weekly"
    
  # バックエンドの依存関係
  - package-ecosystem: "npm"
    directory: "/apps/backend"
    schedule:
      interval: "weekly"
```

### Renovate による自動更新（代替案）

```json
// renovate.json
{
  "extends": ["config:base"],
  "packageRules": [
    {
      "matchUpdateTypes": ["minor", "patch"],
      "automerge": true
    }
  ],
  "rangeStrategy": "bump",
  "lockFileMaintenance": {
    "enabled": true,
    "schedule": ["before 3am on Monday"]
  }
}
```

## セキュリティ

### 脆弱性のスキャン

```bash
# npm audit
npm audit

# ワークスペースごとに実行
npm audit --workspaces

# 自動修正
npm audit fix

# 強制的に修正（破壊的変更を含む）
npm audit fix --force
```

### GitHub Advanced Security

```yaml
# .github/workflows/codeql.yml
name: CodeQL

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Initialize CodeQL
        uses: github/codeql-action/init@v2
        with:
          languages: javascript, typescript
      
      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v2
```

## パフォーマンス最適化

### 依存関係のインストール高速化

```yaml
# GitHub Actions での最適化
- name: Cache dependencies
  uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

### 不要な依存関係の削除

```bash
# 使用されていない依存関係を検出
npx depcheck

# 特定のワークスペースで実行
npx depcheck apps/frontend
```

## トラブルシューティング

### ホイスティングの問題

```bash
# node_modules を削除して再インストール
rm -rf node_modules
rm -rf apps/*/node_modules
rm -rf packages/*/node_modules
npm install
```

### バージョンの競合

```bash
# 依存関係ツリーを確認
npm list <package-name>

# 特定のバージョンを強制
npm install <package-name>@<version> --force
```

### ワークスペースが認識されない

```json
// package.json で workspaces が正しく設定されているか確認
{
  "workspaces": [
    "apps/*",
    "packages/*"
  ]
}
```

## ベストプラクティス

### 1. 依存関係の最小化
- 必要な依存関係のみをインストール
- バンドルサイズを意識

### 2. バージョンの固定
- 本番環境では正確なバージョンを指定
- 開発環境では範囲指定を使用

### 3. 定期的な更新
- 週次または月次で依存関係を更新
- セキュリティパッチは即座に適用

### 4. ドキュメント化
- 重要な依存関係の理由を記録
- バージョン固定の理由を記録

### 5. テストの徹底
- 依存関係更新後は必ずテスト実行
- E2Eテストで破壊的変更を検出

---

**最終更新日**: 2025年12月  
**バージョン**: 1.0.0
