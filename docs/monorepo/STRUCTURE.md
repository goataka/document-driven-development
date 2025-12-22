# モノレポ構成 - ディレクトリ構造

このドキュメントは、モノレポのディレクトリ構造と各ディレクトリの役割を説明します。

## ルートディレクトリ構成

```
spec-driven-development/
├── .github/                    # GitHub設定
│   ├── workflows/              # GitHub Actions
│   ├── CODEOWNERS              # コードオーナー設定
│   └── dependabot.yml          # Dependabot設定
├── apps/                       # アプリケーション
│   ├── frontend/               # フロントエンドアプリ
│   ├── backend/                # バックエンドAPI
│   └── mobile/                 # モバイルアプリ（将来）
├── packages/                   # 共有パッケージ
│   ├── ui/                     # UIコンポーネント
│   ├── types/                  # TypeScript型定義
│   ├── utils/                  # ユーティリティ関数
│   ├── config/                 # 共通設定
│   └── eslint-config/          # ESLint設定
├── sites/               # website
│   ├── user-docs/              # ユーザーマニュアル
│   └── release-notes/          # リリースノート
├── docs/                       # プロジェクトドキュメント
│   ├── agreement/              # 合意事項
│   ├── implement/              # 実装仕様
│   ├── test/                   # テスト戦略
│   ├── build/                  # ビルド戦略
│   ├── deploy/                 # デプロイ戦略
│   ├── operate/                # 運用管理
│   ├── monitor/                # 監視戦略
│   ├── release/                # リリース管理
│   ├── user/                   # ユーザー向け
│   └── monorepo/               # モノレポ構成（このディレクトリ）
├── tools/                      # 開発ツール
│   ├── scripts/                # スクリプト
│   └── generators/             # コード生成
├── .gitignore                  # Git除外設定
├── package.json                # ルートpackage.json
├── turbo.json                  # Turborepo設定
├── tsconfig.base.json          # 共通TypeScript設定
├── .eslintrc.js                # 共通ESLint設定
├── .prettierrc                 # Prettier設定
├── README.md                   # プロジェクトREADME
├── CONTRIBUTING.md             # コントリビューションガイド
└── LICENSE                     # ライセンス
```

## apps/ - アプリケーション

独立してデプロイ可能なアプリケーション。

### apps/frontend/

```
apps/frontend/
├── src/
│   ├── components/             # Reactコンポーネント
│   ├── pages/                  # ページコンポーネント
│   ├── hooks/                  # カスタムフック
│   ├── contexts/               # Reactコンテキスト
│   ├── services/               # APIサービス
│   ├── utils/                  # ユーティリティ（アプリ固有）
│   ├── types/                  # 型定義（アプリ固有）
│   ├── styles/                 # スタイル
│   └── App.tsx                 # アプリエントリーポイント
├── public/                     # 静的ファイル
├── package.json
├── tsconfig.json
├── vite.config.ts              # Vite設定
└── README.md
```

**依存関係**:
- `@repo/ui` - 共通UIコンポーネント
- `@repo/types` - 共有型定義
- `@repo/utils` - 共通ユーティリティ

### apps/backend/

```
apps/backend/
├── src/
│   ├── modules/                # 機能モジュール
│   │   ├── auth/               # 認証モジュール
│   │   ├── attendance/         # 勤怠モジュール
│   │   └── users/              # ユーザーモジュール
│   ├── common/                 # 共通コード
│   │   ├── decorators/         # デコレータ
│   │   ├── filters/            # 例外フィルター
│   │   ├── guards/             # ガード
│   │   ├── interceptors/       # インターセプター
│   │   └── pipes/              # パイプ
│   ├── config/                 # 設定
│   ├── database/               # データベース
│   └── main.ts                 # アプリエントリーポイント
├── test/                       # テスト
├── package.json
├── tsconfig.json
├── nest-cli.json               # NestJS設定
└── README.md
```

**依存関係**:
- `@repo/types` - 共有型定義
- `@repo/utils` - 共通ユーティリティ
- `@repo/config` - 共通設定

## packages/ - 共有パッケージ

複数のアプリで使用する共通コード。

### packages/ui/

```
packages/ui/
├── src/
│   ├── components/             # UIコンポーネント
│   │   ├── Button/
│   │   ├── Input/
│   │   ├── Modal/
│   │   └── index.ts
│   ├── hooks/                  # カスタムフック
│   ├── styles/                 # スタイル
│   └── index.ts                # エクスポート
├── package.json
├── tsconfig.json
└── README.md
```

### packages/types/

```
packages/types/
├── src/
│   ├── api/                    # API型定義
│   │   ├── auth.ts
│   │   ├── attendance.ts
│   │   └── users.ts
│   ├── models/                 # データモデル
│   │   ├── User.ts
│   │   └── Attendance.ts
│   ├── enums/                  # Enum定義
│   └── index.ts
├── package.json
├── tsconfig.json
└── README.md
```

### packages/utils/

```
packages/utils/
├── src/
│   ├── date/                   # 日付ユーティリティ
│   ├── string/                 # 文字列ユーティリティ
│   ├── validation/             # バリデーション
│   └── index.ts
├── test/                       # ユニットテスト
├── package.json
├── tsconfig.json
└── README.md
```

### packages/config/

```
packages/config/
├── src/
│   ├── env/                    # 環境変数設定
│   ├── constants/              # 定数
│   └── index.ts
├── package.json
├── tsconfig.json
└── README.md
```

### packages/eslint-config/

```
packages/eslint-config/
├── index.js                    # 共通ESLint設定
├── react.js                    # React用設定
├── node.js                     # Node.js用設定
├── package.json
└── README.md
```

## sites/ - website

websiteジェネレーターで生成するサイト。

### sites/user-docs/

```
sites/user-docs/
├── mkdocs.yml                  # MkDocs設定
├── docs/
│   ├── index.md
│   ├── guides/
│   ├── reference/
│   └── images/
├── site/                       # ビルド成果物（Git除外）
└── README.md
```

### sites/release-notes/

```
sites/release-notes/
├── mkdocs.yml
├── docs/
│   ├── index.md
│   └── versions/
├── site/                       # ビルド成果物（Git除外）
└── README.md
```

## tools/ - 開発ツール

開発やビルドに使用するツール・スクリプト。

### tools/scripts/

```
tools/scripts/
├── build.sh                    # ビルドスクリプト
├── deploy.sh                   # デプロイスクリプト
├── setup.sh                    # セットアップスクリプト
└── check-deps.ts               # 依存関係チェック
```

### tools/generators/

```
tools/generators/
├── component/                  # コンポーネント生成
├── module/                     # モジュール生成
└── package/                    # パッケージ生成
```

## ルート設定ファイル

### package.json

```json
{
  "name": "spec-driven-development",
  "version": "1.0.0",
  "private": true,
  "workspaces": [
    "apps/*",
    "packages/*",
    "sites/*"
  ],
  "scripts": {
    "build": "turbo run build",
    "dev": "turbo run dev --parallel",
    "test": "turbo run test",
    "lint": "turbo run lint",
    "format": "prettier --write \"**/*.{ts,tsx,md}\""
  },
  "devDependencies": {
    "turbo": "^1.10.0",
    "prettier": "^3.0.0",
    "typescript": "^5.0.0"
  }
}
```

### turbo.json

```json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**", "build/**"]
    },
    "test": {
      "dependsOn": ["build"],
      "outputs": ["coverage/**"]
    },
    "lint": {
      "outputs": []
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

### tsconfig.base.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "jsx": "react-jsx",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "moduleResolution": "node",
    "paths": {
      "@repo/ui": ["./packages/ui/src"],
      "@repo/types": ["./packages/types/src"],
      "@repo/utils": ["./packages/utils/src"],
      "@repo/config": ["./packages/config/src"]
    }
  }
}
```

## .gitignore

```gitignore
# 依存関係
node_modules/
.pnp
.pnp.js

# ビルド成果物
dist/
build/
.next/
out/
site/

# 環境変数
.env
.env.local
.env.*.local

# ログ
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# IDE
.idea/
.vscode/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# テスト
coverage/
.nyc_output/

# Turborepo
.turbo/
```

## CODEOWNERS

```
# デフォルトオーナー
* @team-lead

# フロントエンド
/apps/frontend/ @frontend-team
/packages/ui/ @frontend-team

# バックエンド
/apps/backend/ @backend-team

# ドキュメント
/docs/ @tech-writers
/sites/ @tech-writers

# 設定ファイル
*.json @team-lead
*.yml @team-lead
*.yaml @team-lead
```

## パッケージのネーミング規則

### スコープ付きパッケージ

```json
{
  "name": "@repo/ui",
  "version": "1.0.0"
}
```

### アプリケーション

```json
{
  "name": "@apps/frontend",
  "version": "1.0.0",
  "private": true
}
```

## ディレクトリの追加ルール

### 新しいアプリの追加

1. `apps/` 配下に新規ディレクトリ作成
2. `package.json` を作成
3. `README.md` を作成
4. `turbo.json` にタスク追加（必要に応じて）

### 新しいパッケージの追加

1. `packages/` 配下に新規ディレクトリ作成
2. `package.json` を作成（スコープ: `@repo/`）
3. `src/index.ts` を作成
4. `README.md` を作成
5. 他のパッケージから参照可能に

---

**最終更新日**: 2025年12月  
**バージョン**: 1.0.0
