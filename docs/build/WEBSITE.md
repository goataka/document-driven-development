# websiteビルド方針

本ドキュメントは、ドキュメントwebsite（リリースノート、ユーザーマニュアル）のビルド方針を説明します。

## 概要

ユーザー向けドキュメントとリリースノートをwebsiteとして提供します。

### 対象ドキュメント

- **ユーザーマニュアル**: `docs/user/` 配下のMarkdownファイル
- **リリースノート**: `docs/user/RELEASE_NOTES.md`
- **FAQ**: `docs/user/FAQ.md`

### 基本方針

- Markdown形式でドキュメントを作成
- websiteジェネレーターでHTMLに変換
- 認証なしで公開アクセス可能
- アプリケーション機能内からリンク

---

## websiteジェネレーター

### 1. MkDocs（推奨）

**特徴**:
- Python製、シンプルで使いやすい
- Material for MkDocsテーマで美しいデザイン
- 日本語検索対応
- レスポンシブデザイン

**設定例**:
```yaml
# mkdocs.yml
site_name: 勤怠管理システム - ドキュメント
site_url: https://example.github.io/attendance-system/
repo_url: https://github.com/example/attendance-system

theme:
  name: material
  language: ja
  features:
    - navigation.tabs
    - navigation.sections
    - navigation.expand
    - navigation.top
    - search.suggest
    - search.highlight
    - content.code.copy

nav:
  - ホーム: index.md
  - スタートアップガイド: guides/startup.md
  - 機能リファレンス:
    - ユーザー登録: reference/user-registration.md
    - ログイン: reference/login.md
    - 出勤・退勤打刻: reference/clock-in-out.md
    - 勤怠履歴: reference/attendance-history.md
  - トラブルシューティング: guides/troubleshooting.md
  - FAQ: FAQ.md
  - リリースノート: RELEASE_NOTES.md

markdown_extensions:
  - admonition
  - codehilite
  - toc:
      permalink: true
```

### 2. VitePress

**特徴**:
- Vue.js製、高速でモダン
- 開発者向けドキュメントに最適
- ホットリロード対応

**設定例**:
```javascript
// docs/.vitepress/config.js
export default {
  title: '勤怠管理システム',
  description: 'ユーザーマニュアルとドキュメント',
  lang: 'ja',
  
  themeConfig: {
    nav: [
      { text: 'ホーム', link: '/' },
      { text: 'ガイド', link: '/guides/startup' },
      { text: 'リファレンス', link: '/reference/' },
      { text: 'FAQ', link: '/FAQ' }
    ],
    
    sidebar: {
      '/guides/': [
        {
          text: 'ガイド',
          items: [
            { text: 'スタートアップガイド', link: '/guides/startup' },
            { text: 'トラブルシューティング', link: '/guides/troubleshooting' }
          ]
        }
      ],
      '/reference/': [
        {
          text: '機能リファレンス',
          items: [
            { text: 'ユーザー登録', link: '/reference/user-registration' },
            { text: 'ログイン', link: '/reference/login' },
            { text: '出勤・退勤打刻', link: '/reference/clock-in-out' },
            { text: '勤怠履歴', link: '/reference/attendance-history' }
          ]
        }
      ]
    }
  }
}
```

---

## ビルドプロセス

### ローカルビルド

#### MkDocsの場合

```bash
# インストール
pip install mkdocs-material

# 開発サーバー起動
mkdocs serve

# ビルド
mkdocs build

# 出力: site/ ディレクトリ
```

#### VitePressの場合

```bash
# インストール
npm install -D vitepress

# 開発サーバー起動
npm run docs:dev

# ビルド
npm run docs:build

# 出力: docs/.vitepress/dist/ ディレクトリ
```

### CI/CDビルド

GitHub Actionsでの自動ビルド例:

```yaml
# .github/workflows/build-docs.yml
name: Build Documentation

on:
  push:
    branches:
      - main
    paths:
      - 'docs/**'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.x'
      
      - name: Install MkDocs
        run: |
          pip install mkdocs-material
      
      - name: Build docs
        run: mkdocs build
      
      - name: Upload artifact
        uses: actions/upload-artifact@v3
        with:
          name: documentation-site
          path: site/
```

---

## ディレクトリ構成

### 推奨構成（MkDocs）

```
sites/user-docs/
├── mkdocs.yml              # MkDocs設定ファイル
├── docs/
│   ├── index.md            # トップページ
│   ├── guides/
│   │   ├── startup.md      # スタートアップガイド
│   │   └── troubleshooting.md  # トラブルシューティング
│   ├── reference/
│   │   ├── user-registration.md
│   │   ├── login.md
│   │   ├── logout.md
│   │   ├── clock-in-out.md
│   │   └── attendance-history.md
│   ├── FAQ.md              # よくある質問
│   ├── RELEASE_NOTES.md    # リリースノート
│   └── images/             # 画像ファイル
│       ├── login-screen.png
│       ├── dashboard.png
│       └── ...
└── site/                   # ビルド後の静的ファイル（自動生成）
```

### 推奨構成（VitePress）

```
sites/user-docs/
├── package.json
├── docs/
│   ├── .vitepress/
│   │   ├── config.js       # VitePress設定
│   │   └── theme/          # カスタムテーマ
│   ├── index.md
│   ├── guides/
│   ├── reference/
│   ├── FAQ.md
│   ├── RELEASE_NOTES.md
│   └── public/             # 静的アセット
│       └── images/
└── dist/                   # ビルド後の静的ファイル（自動生成）
```

---

## ビルド最適化

### 画像の最適化

```bash
# WebP形式への変換
cwebp input.png -o output.webp

# 画像の圧縮
optipng -o7 *.png
jpegoptim --max=85 *.jpg
```

### ビルドキャッシュ

```yaml
# GitHub Actionsでのキャッシュ例
- name: Cache pip packages
  uses: actions/cache@v3
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}
```

### 静的ファイルの圧縮

```yaml
# ビルド後の圧縮
- name: Compress static files
  run: |
    find site -name "*.html" -exec gzip -k {} \;
    find site -name "*.css" -exec gzip -k {} \;
    find site -name "*.js" -exec gzip -k {} \;
```

---

## テーマカスタマイズ

### MkDocs Materialテーマ

```yaml
# mkdocs.yml
theme:
  name: material
  palette:
    # ライトモード
    - media: "(prefers-color-scheme: light)"
      scheme: default
      primary: indigo
      accent: indigo
      toggle:
        icon: material/brightness-7
        name: ダークモードに切り替え
    
    # ダークモード
    - media: "(prefers-color-scheme: dark)"
      scheme: slate
      primary: indigo
      accent: indigo
      toggle:
        icon: material/brightness-4
        name: ライトモードに切り替え

extra_css:
  - stylesheets/extra.css
```

```css
/* docs/stylesheets/extra.css */
.md-header {
  background-color: #1976d2;
}

.md-typeset h1 {
  color: #1976d2;
}
```

---

## パフォーマンス

### ビルド時間の短縮

- 不要なMarkdownファイルを除外
- プラグインの最小化
- 並列ビルドの活用

### サイト表示速度

- 静的ファイルの圧縮（gzip）
- 画像の最適化（WebP形式）
- CDN経由での配信

---

## トラブルシューティング

### ビルドエラー

```bash
# ローカルでビルドテスト
mkdocs build --strict

# エラーログを確認
mkdocs build --verbose
```

### リンク切れチェック

```bash
# リンクチェックツール
npm install -g broken-link-checker
blc https://your-docs-site.com -ro
```

---

## 次のステップ

- [デプロイ戦略](../deploy/STATIC_SITE.md) - ホスティング先とデプロイ方法
- [アプリ連携](../implement/STATIC_SITE.md) - アプリケーションからのリンク方法

---

**最終更新日**: 2025年12月  
**バージョン**: 1.0.0
