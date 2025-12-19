# セキュリティ・脆弱性テスト (静的解析)

本ドキュメントは、静的セキュリティテストと脆弱性スキャンの実装方法を説明します。
**すべて無料のツールを使用しています。**

**関連ドキュメント**: 
- [テスト戦略概要](./OVERVIEW.md)
- [動的セキュリティテスト](./SECURITY_DYNAMIC.md)
- [システムアーキテクチャ設計書](../implement/ARCHITECTURE.md)
- [セキュリティ設計書](../implement/SECURITY.md)

## 目次

1. [静的セキュリティテスト](#静的セキュリティテスト)
2. [ツール](#ツール)
3. [セキュリティチェックリスト](#セキュリティチェックリスト)
4. [ベストプラクティス](#ベストプラクティス)

---

## 静的セキュリティテスト

静的セキュリティテストは、コードとその依存関係の脆弱性を検出します。

### npm audit

定期的に依存関係の脆弱性をチェックします。

```bash
# 脆弱性スキャン
npm audit

# 自動修正（メジャーバージョンは除く）
npm audit fix

# 全ての修正を適用
npm audit fix --force
```

### audit-ci

CI/CDで脆弱性チェックを強制するために、audit-ciを使用します（無料）。

#### インストールと設定

```bash
# audit-ci のインストール
npm install --save-dev audit-ci
```

#### package.json に追加

```json
{
  "scripts": {
    "audit:check": "audit-ci --moderate"
  }
}
```

#### CI/CD統合

```yaml
# .github/workflows/security.yml
name: Security Scan

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]
  schedule:
    # 毎日午前2時に実行
    - cron: '0 2 * * *'

jobs:
  security:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run npm audit
        run: npm audit --audit-level=moderate
      
      - name: Run audit-ci
        run: npx audit-ci --moderate
      
      - name: Dependency Review
        uses: actions/dependency-review-action@v4
        if: github.event_name == 'pull_request'
```

### GitHub CodeQL (無料)

GitHub の組み込みセキュリティスキャン機能を活用します。

```yaml
# .github/workflows/codeql.yml
name: CodeQL Analysis

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]
  schedule:
    - cron: '0 2 * * 1'

jobs:
  analyze:
    name: Analyze
    runs-on: ubuntu-latest
    permissions:
      actions: read
      contents: read
      security-events: write

    strategy:
      fail-fast: false
      matrix:
        language: ['javascript', 'typescript']

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}

      - name: Autobuild
        uses: github/codeql-action/autobuild@v3

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3
```

---

## セキュリティチェックリスト

- ✅ 全てのユーザー入力をバリデーション
- ✅ SQLインジェクション対策（パラメータ化クエリ使用）
- ✅ XSS対策（出力エスケープ）
- ✅ CSRF対策（トークン検証）
- ✅ 適切な認証・認可実装
- ✅ パスワードの安全なハッシュ化（bcrypt）
- ✅ HTTPS通信の強制
- ✅ セキュリティヘッダーの設定（Helmet使用）
- ✅ レート制限の実装
- ✅ 定期的な依存関係の更新と脆弱性スキャン

---

## ベストプラクティス

1. **セキュリティファースト**
   - 設計段階からセキュリティを考慮
   - 新機能ごとにセキュリティテスト

2. **定期的なスキャン**
   - 週次で脆弱性スキャン実行
   - 依存関係は常に最新に

3. **脆弱性は即修正**
   - Critical/High は即対応
   - Medium は1週間以内に対応

4. **セキュリティ教育**
   - チーム全体でセキュリティ意識向上
   - OWASP Top 10を理解

---

## 関連ドキュメント

- [テスト戦略概要](./OVERVIEW.md)
- [動的セキュリティテスト](./SECURITY_DYNAMIC.md)
- [統合・E2Eテスト](./INTEGRATION.md)
- [依存関係管理](../implement/DEPENDENCIES.md)
- [セキュリティ設計書](../implement/SECURITY.md)
