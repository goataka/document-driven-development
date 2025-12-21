# 静的サイトデプロイ方針

本ドキュメントは、ドキュメント静的サイトのホスティングとデプロイ方法を説明します。

## 概要

ユーザーマニュアルとリリースノートを静的サイトとして提供します。

### 基本方針

- 認証なしで誰でもアクセス可能
- HTTPS対応必須
- 高速な表示（CDN活用）
- 自動デプロイ（CI/CD）

---

## ホスティングオプション

### 1. GitHub Pages（推奨）

**メリット**:
- 完全無料
- GitHubリポジトリと統合
- 自動デプロイ可能
- HTTPS対応

**公開URL**: `https://<username>.github.io/<repository>/`

**設定方法**:
```yaml
# .github/workflows/deploy-docs.yml
name: Deploy Documentation

on:
  push:
    branches:
      - main
    paths:
      - 'docs/**'
      - 'static-sites/**'

jobs:
  deploy:
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
        run: |
          cd static-sites/user-docs
          mkdocs build
      
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./static-sites/user-docs/site
```

**カスタムドメイン設定**:
1. リポジトリ設定から「Pages」セクションへ移動
2. 「Custom domain」にドメインを入力
3. DNSレコードを設定:
   ```
   CNAME docs.example.com -> <username>.github.io
   ```

### 2. Vercel

**メリット**:
- 高速なCDN
- プレビュー環境自動生成
- カスタムドメイン対応
- 無料プランあり

**公開URL**: `https://<project-name>.vercel.app`

**設定方法**:
```json
// vercel.json
{
  "buildCommand": "cd static-sites/user-docs && mkdocs build",
  "outputDirectory": "static-sites/user-docs/site",
  "devCommand": "cd static-sites/user-docs && mkdocs serve"
}
```

### 3. Netlify

**メリット**:
- 簡単なデプロイ設定
- フォームやリダイレクト機能
- プレビューデプロイ
- 無料プランあり

**公開URL**: `https://<site-name>.netlify.app`

**設定方法**:
```toml
# netlify.toml
[build]
  command = "cd static-sites/user-docs && mkdocs build"
  publish = "static-sites/user-docs/site"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

### 4. AWS S3 + CloudFront

**メリット**:
- AWSエコシステムとの統合
- 高度なカスタマイズ可能
- 企業向け運用に適している

**デメリット**:
- 初期設定が複雑
- コストがかかる

**概算コスト**: 月額 $1-5（トラフィック次第）

---

## デプロイフロー

### 自動デプロイ

```mermaid
graph LR
    A[Markdownを更新] --> B[GitHubにプッシュ]
    B --> C[GitHub Actions起動]
    C --> D[静的サイト生成]
    D --> E[ホスティング先へデプロイ]
    E --> F[サイト更新完了]
```

### 手動デプロイ

```bash
# MkDocsの場合
cd static-sites/user-docs
mkdocs build
mkdocs gh-deploy

# VitePressの場合
cd static-sites/user-docs
npm run docs:build
# 生成されたdist/をホスティング先へアップロード
```

---

## 環境別デプロイ

### 開発環境
- **ブランチ**: `develop`
- **URL**: `https://dev-docs.example.com`
- **用途**: 開発中のドキュメント確認

### ステージング環境
- **ブランチ**: `staging`
- **URL**: `https://staging-docs.example.com`
- **用途**: リリース前の最終確認

### 本番環境
- **ブランチ**: `main`
- **URL**: `https://docs.example.com`
- **用途**: ユーザーに公開

---

## デプロイ監視

### デプロイステータスの確認

```yaml
# GitHub Actions バッジ
![Deploy Status](https://github.com/<user>/<repo>/workflows/Deploy%20Documentation/badge.svg)
```

### デプロイ通知

Slack通知の設定例:
```yaml
- name: Notify Slack
  if: always()
  uses: 8398a7/action-slack@v3
  with:
    status: ${{ job.status }}
    text: 'ドキュメントのデプロイが完了しました'
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

---

## セキュリティ

### 公開情報の管理
- 機密情報は含めない
- APIキーやパスワードは除外
- 内部システムの詳細は公開しない

### HTTPS必須
- すべてのホスティングでHTTPSを有効化
- mixed contentエラーを防ぐ

### アクセス制限（必要な場合）

**Netlify基本認証**:
```toml
[[headers]]
  for = "/*"
  [headers.values]
    Basic-Auth = "username:password"
```

**Vercel認証**:
```json
{
  "password": {
    "mode": "basic",
    "username": "admin",
    "password": "secure-password"
  }
}
```

---

## 運用とメンテナンス

### 定期的な更新
- 週次または月次でドキュメントレビュー
- 古い情報の更新
- リンク切れの修正

### バージョン管理

**mikeを使用したバージョニング**:
```yaml
# mkdocs.yml
extra:
  version:
    provider: mike
    default: latest

plugins:
  - mike:
      version_selector: true
```

デプロイコマンド：
```bash
# バージョン 1.0 をデプロイ
mike deploy 1.0 latest --update-aliases

# バージョン一覧
mike list
```

### アクセス分析

**Google Analytics導入**:
```yaml
# mkdocs.yml
extra:
  analytics:
    provider: google
    property: G-XXXXXXXXXX
```

---

## パフォーマンス最適化

### CDN活用
- CloudFront（AWS）
- Fastly（Netlify）
- Vercel Edge Network

### キャッシュ戦略

```yaml
# Netlifyキャッシュ設定
[[headers]]
  for = "/*.html"
  [headers.values]
    Cache-Control = "public, max-age=0, must-revalidate"

[[headers]]
  for = "/assets/*"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"
```

### 静的ファイル圧縮

```bash
# ビルド後の圧縮
find site -name "*.html" -exec gzip -k {} \;
find site -name "*.css" -exec gzip -k {} \;
find site -name "*.js" -exec gzip -k {} \;
```

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

### キャッシュクリア

```yaml
# GitHub Actionsでキャッシュをクリア
- name: Clear cache
  run: |
    rm -rf ~/.cache/pip
    pip cache purge
```

---

## 次のステップ

- [ビルド方針](../build/STATIC_SITE.md) - 静的サイトのビルド設定
- [アプリ連携](../implement/STATIC_SITE.md) - アプリケーションからのリンク方法

---

**最終更新日**: 2025年12月  
**バージョン**: 1.0.0
