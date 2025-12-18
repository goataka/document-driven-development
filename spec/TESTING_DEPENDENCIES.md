# 依存関係管理と自動更新

本ドキュメントは、依存関係の管理と自動更新の設定方法を説明します。
**すべて無料のツールを使用しています。**

**関連ドキュメント**: 
- [テスト戦略概要](./TESTING.md)
- [システムアーキテクチャ設計書](./ARCHITECTURE.md)

## 目次

1. [依存関係管理と自動更新](#依存関係管理と自動更新)
2. [Dependabot](#dependabot)
3. [Renovate](#renovate)
4. [依存関係チェックのワークフロー](#依存関係チェックのワークフロー)
5. [npm-check-updates](#npm-check-updates)
6. [ベストプラクティス](#ベストプラクティス)

---

## 依存関係管理と自動更新

依存関係を最新かつ安全な状態に保つための戦略とツールです。

### Dependabot（無料：GitHub標準機能）

GitHub Dependabotを使用して、依存関係の自動更新を設定します。パブリック・プライベートリポジトリともに無料で利用可能です。

#### 設定ファイル

```yaml
# .github/dependabot.yml
version: 2
updates:
  # npm dependencies (frontend)
  - package-ecosystem: "npm"
    directory: "/frontend"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
      timezone: "Asia/Tokyo"
    open-pull-requests-limit: 10
    reviewers:
      - "team-developers"
    assignees:
      - "tech-lead"
    labels:
      - "dependencies"
      - "frontend"
    commit-message:
      prefix: "chore(deps):"
    # セキュリティアップデートは即座にマージ
    # 通常のアップデートはレビュー後にマージ
    versioning-strategy: increase
    
  # npm dependencies (backend)
  - package-ecosystem: "npm"
    directory: "/backend"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
      timezone: "Asia/Tokyo"
    open-pull-requests-limit: 10
    reviewers:
      - "team-developers"
    assignees:
      - "tech-lead"
    labels:
      - "dependencies"
      - "backend"
    commit-message:
      prefix: "chore(deps):"
    versioning-strategy: increase
    
  # GitHub Actions
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
      timezone: "Asia/Tokyo"
    labels:
      - "dependencies"
      - "ci-cd"
    commit-message:
      prefix: "chore(ci):"
```

### Renovate（無料：オープンソース・セルフホスト可能）

Renovate は Dependabot よりも柔軟な設定が可能です。オープンソースプロジェクトでは無料、セルフホストも可能です。

```json
// renovate.json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:base"],
  "schedule": ["every weekend"],
  "timezone": "Asia/Tokyo",
  "labels": ["dependencies"],
  "packageRules": [
    {
      "matchUpdateTypes": ["minor", "patch"],
      "matchCurrentVersion": "!/^0/",
      "automerge": true,
      "automergeType": "pr",
      "automergeStrategy": "squash"
    },
    {
      "matchDepTypes": ["devDependencies"],
      "matchUpdateTypes": ["minor", "patch"],
      "automerge": true
    },
    {
      "matchPackagePatterns": ["^@types/"],
      "automerge": true
    },
    {
      "matchPackageNames": ["typescript", "eslint", "prettier"],
      "groupName": "linting and formatting"
    },
    {
      "matchPackagePatterns": ["^@testing-library/", "^vitest", "^jest"],
      "groupName": "testing tools"
    }
  ],
  "vulnerabilityAlerts": {
    "labels": ["security"],
    "assignees": ["@team-security"]
  }
}
```

### 依存関係チェックのワークフロー

```yaml
# .github/workflows/dependency-check.yml
name: Dependency Check

on:
  schedule:
    # 毎週月曜日午前9時に実行
    - cron: '0 0 * * 1'
  workflow_dispatch:

jobs:
  check-dependencies:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Check for outdated packages
        run: npm outdated || true
      
      - name: Check for security vulnerabilities
        run: npm audit --audit-level=moderate
      
      - name: Run audit-ci
        run: npx audit-ci --moderate
      
      - name: Check for deprecated packages
        run: npx check-dependencies
      
      - name: Generate dependency report
        run: |
          echo "# 依存関係レポート" > dependency-report.md
          echo "## Outdated Packages" >> dependency-report.md
          npm outdated --json >> dependency-report.md || true
          echo "## Security Audit" >> dependency-report.md
          npm audit --json >> dependency-report.md || true
      
      - name: Upload dependency report
        uses: actions/upload-artifact@v4
        with:
          name: dependency-report
          path: dependency-report.md
```

### npm-check-updates

手動で依存関係を更新する場合に便利なツールです。

```bash
# npm-check-updates のインストール
npm install -g npm-check-updates

# 更新可能なパッケージを確認
ncu

# 全てのパッケージを最新版に更新
ncu -u

# インストール
npm install

# テスト実行
npm test
```

### 依存関係管理のベストプラクティス

1. **定期的な更新**: 週次でDependabotによる自動更新を実行
2. **セキュリティパッチ優先**: セキュリティアップデートは即座に適用
3. **グループ化**: 関連する依存関係をグループ化して一括更新
4. **自動マージ**: パッチ・マイナーバージョンの更新は自動マージ
5. **メジャーバージョン**: 手動レビュー必須
6. **テスト必須**: CI/CDでの自動テストをパス後にマージ
7. **ロックファイルのコミット**: `package-lock.json` を必ずコミット
8. **監査ログ**: 更新履歴とテスト結果を記録

### package.json のバージョン管理戦略

```json
{
  "dependencies": {
    // ピン留め（推奨：本番環境の重要なライブラリ）
    "react": "18.2.0",
    
    // マイナー・パッチ更新許可（推奨：安定したライブラリ）
    "axios": "^1.6.0",
    
    // パッチ更新のみ許可（慎重な場合）
    "lodash": "~4.17.21"
  },
  "devDependencies": {
    // 開発ツールは柔軟に更新可能
    "typescript": "^5.3.0",
    "vitest": "^1.0.0"
  }
}
```

---



---

## ベストプラクティス

1. **定期的な更新**
   - 週次でDependabotをチェック
   - セキュリティアップデートは即適用

2. **テスト後にマージ**
   - 自動テストが通過したらマージ
   - Breaking changesは慎重に

3. **バージョン管理戦略**
   - 本番: 固定バージョン（^なし）
   - 開発: マイナー・パッチ自動更新

4. **依存関係の最小化**
   - 不要なパッケージは削除
   - 軽量な代替を検討

5. **ロックファイルをコミット**
   - package-lock.json必須
   - 環境を一致させる

6. **監査を習慣化**
   - 毎回npm install時にaudit
   - CI/CDで強制チェック

7. **ドキュメント化**
   - 依存関係の目的を記録
   - 更新履歴を残す

8. **セキュリティアラートを有効化**
   - GitHub Security advisories
   - すぐに通知を受け取る

---

## 関連ドキュメント

- [テスト戦略概要](./TESTING.md)
- [統合・E2Eテスト](./TESTING_INTEGRATION.md)
- [品質テスト](./TESTING_QUALITY.md)
- [セキュリティ・脆弱性テスト](./TESTING_SECURITY.md)
