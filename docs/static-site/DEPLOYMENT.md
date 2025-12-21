# 静的サイト - デプロイ戦略

このドキュメントは、静的サイトのホスティングとデプロイ方法を説明します。

## ホスティングオプション

### 1. GitHub Pages（推奨）

**メリット**:
- 完全無料
- GitHubリポジトリと統合
- 自動デプロイ可能
- HTTPS対応

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
        run: mkdocs build
      
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./site
```

**公開URL**: `https://<username>.github.io/<repository>/`

### 2. Vercel

**メリット**:
- 高速なCDN
- プレビュー環境自動生成
- カスタムドメイン対応
- 無料プランあり

**設定方法**:
```json
// vercel.json
{
  "buildCommand": "mkdocs build",
  "outputDirectory": "site",
  "devCommand": "mkdocs serve"
}
```

**公開URL**: `https://<project-name>.vercel.app`

### 3. Netlify

**メリット**:
- 簡単なデプロイ設定
- フォームやリダイレクト機能
- プレビューデプロイ
- 無料プランあり

**設定方法**:
```toml
# netlify.toml
[build]
  command = "mkdocs build"
  publish = "site"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

**公開URL**: `https://<site-name>.netlify.app`

### 4. AWS S3 + CloudFront

**メリット**:
- AWSエコシステムとの統合
- 高度なカスタマイズ可能
- 企業向け運用に適している

**デメリット**:
- 初期設定が複雑
- コストがかかる

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
mkdocs build
mkdocs gh-deploy

# VitePressの場合
npm run docs:build
# 生成されたdist/をホスティング先へアップロード
```

## デプロイ設定

### 1. MkDocsの設定

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

### 2. VitePressの設定

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

## 環境別デプロイ

### 開発環境
- ブランチ: `develop`
- URL: `https://dev-docs.example.com`
- 用途: 開発中のドキュメント確認

### ステージング環境
- ブランチ: `staging`
- URL: `https://staging-docs.example.com`
- 用途: リリース前の最終確認

### 本番環境
- ブランチ: `main`
- URL: `https://docs.example.com`
- 用途: ユーザーに公開

## カスタムドメイン設定

### GitHub Pagesの場合

1. リポジトリ設定から「Pages」セクションへ移動
2. 「Custom domain」にドメインを入力
3. DNSレコードを設定:
   ```
   CNAME docs.example.com -> <username>.github.io
   ```

### Vercel/Netlifyの場合

1. プロジェクト設定から「Domains」セクションへ移動
2. カスタムドメインを追加
3. DNS設定の指示に従う

## デプロイの監視

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

## セキュリティ考慮事項

### 公開情報の管理
- 機密情報は含めない
- APIキーやパスワードは除外
- 内部システムの詳細は公開しない

### HTTPS必須
- すべてのホスティングでHTTPSを有効化
- mixed contentエラーを防ぐ

### アクセス制限（必要な場合）
- 基本認証の追加（Netlify/Vercel）
- IPアドレス制限（AWS CloudFront）

## 運用とメンテナンス

### 定期的な更新
- 週次または月次でドキュメントレビュー
- 古い情報の更新
- リンク切れの修正

### バージョン管理
- リリースごとにドキュメントをバージョニング
- 古いバージョンのドキュメントも保持

### アクセス分析
- Google Analyticsの導入
- よく閲覧されるページの把握
- 検索キーワードの分析

---

**最終更新日**: 2025年12月  
**バージョン**: 1.0.0
