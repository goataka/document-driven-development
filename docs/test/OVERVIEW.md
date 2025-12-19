# テスト戦略概要

本ドキュメントは、システム全体のテスト戦略の概要を説明します。
**すべて無料のツールを使用しています。**

**関連ドキュメント**: 
- [システムアーキテクチャ設計書](../implement/ARCHITECTURE.md)
- [単体テスト](./UNIT.md)
- [コンポーネントテスト](./COMPONENT.md)
- [統合テスト](./INTEGRATION.md)
- [E2Eテスト](./E2E.md)
- [スナップショットテスト](./SNAPSHOT.md)
- [アクセシビリティテスト](./ACCESSIBILITY.md)
- [パフォーマンステスト](./PERFORMANCE.md)
- [セキュリティテスト](./SECURITY.md)
- [動的セキュリティテスト](./SECURITY_DYNAMIC.md)
- [依存関係管理](../implement/DEPENDENCIES.md)

## 目次

1. [テストピラミッド](#テストピラミッド)
2. [テストの種類](#テストの種類)
3. [テストカバレッジ目標](#テストカバレッジ目標)
4. [CI/CD統合](#cicd統合)
5. [ベストプラクティス](#ベストプラクティス)

---

## テストピラミッド

テストは品質保証の要であり、以下の複数のレイヤーで包括的にテストを実施します。

```
           ┌─────────────────┐
           │   E2Eテスト     │  少数・遅い・高コスト
           │   (Cucumber +   │
           │   Playwright)   │
           └─────────────────┘
                   △
                  ╱ ╲
                 ╱   ╲
                ╱     ╲
               ╱       ╲
         ┌────────────────┐
         │  統合テスト     │   中程度
         │  (Jest)        │
         └────────────────┘
                △
               ╱ ╲
              ╱   ╲
             ╱     ╲
            ╱       ╲
      ┌──────────────────┐
      │   単体テスト      │   多数・速い・低コスト
      │   (Jest/Vitest)  │
      └──────────────────┘
```

---



---

| パフォーマンステスト | 全主要ページ | Core Web Vitals目標達成 |
| セキュリティテスト | 全APIエンドポイント | 脆弱性ゼロ |

---

## CI/CD統合

### GitHub Actions設定例

```yaml
# .github/workflows/test.yml
name: Test

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  unit-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      # フロントエンド単体テスト
      - name: Frontend Unit Tests
        working-directory: ./frontend
        run: |
          npm ci
          npm run test -- --coverage
      
      # バックエンド単体テスト
      - name: Backend Unit Tests
        working-directory: ./backend
        run: |
          npm ci
          npm run test -- --coverage
      
      - name: Upload Coverage
        uses: codecov/codecov-action@v3

  integration-test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Backend Integration Tests
        working-directory: ./backend
        run: |
          npm ci
          npm run test:e2e
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db

  e2e-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install Playwright
        run: npx playwright install --with-deps
      
      - name: Start Backend
        working-directory: ./backend
        env:
          API_URL: http://localhost:3000/api
          HEALTH_CHECK_TIMEOUT: 60000
        run: |
          npm ci
          npm run start:prod &
          # APIサーバーの起動を待機（環境変数で設定可能、クォートで安全に）
          npx wait-on "${API_URL}" --timeout "${HEALTH_CHECK_TIMEOUT}"
      
      - name: Start Frontend
        working-directory: ./frontend
        env:
          FRONTEND_URL: http://localhost:5173
          HEALTH_CHECK_TIMEOUT: 60000
        run: |
          npm ci
          npm run build
          npm run preview &
          # フロントエンドの起動を待機（環境変数で設定可能、クォートで安全に）
          npx wait-on "${FRONTEND_URL}" --timeout "${HEALTH_CHECK_TIMEOUT}"
      
      - name: Run E2E Tests
        run: |
          cd e2e
          npm ci
          npm run test:e2e
      
      - name: Upload Test Results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: e2e/playwright-report/
```

---

## ベストプラクティス

1. **テストの独立性**: 各テストは他のテストに依存せず独立して実行可能であること
2. **データのセットアップ**: テストデータは各テストで準備し、クリーンアップすること
3. **モックの活用**: 外部依存を適切にモック化すること
4. **明確なテスト名**: テストケース名は何をテストしているか明確にすること
5. **AAA パターン**: Arrange（準備）、Act（実行）、Assert（検証）の順で書くこと
6. **data-testid属性の使用**: E2Eテストでは`data-testid`属性を使用して安定したセレクタを実現すること（国際化対応にも有効）
7. **null安全性**: TypeScriptのnon-null assertion operator (`!`) を避け、適切なnullチェックを行うこと
8. **CI/CD統合**: 全テストがCI/CDパイプラインで自動実行されること
9. **レポート**: テスト結果とカバレッジレポートを可視化すること
10. **スナップショットレビュー**: スナップショット更新時は差分を必ず確認すること
11. **アクセシビリティ優先**: 全UIコンポーネントでアクセシビリティテストを実施すること
12. **パフォーマンス監視**: 定期的にパフォーマンステストを実行し、劣化を早期検出すること
13. **セキュリティファースト**: セキュリティテストを開発サイクルに組み込むこと
14. **依存関係の最新化**: 定期的に依存関係を更新し、脆弱性を解消すること
15. **テスト自動化**: 手動テストを最小限にし、自動化可能なテストは全て自動化すること

---

**関連ドキュメント**:
- [システムアーキテクチャ設計書](./ARCHITECTURE.md)
- [フロントエンド設計](./FRONTEND.md)
- [バックエンド設計](./BACKEND.md)

---

**最終更新日**: 2024年12月18日  
**バージョン**: 2.0.0


---

## 関連ドキュメント

- [システムアーキテクチャ設計書](./ARCHITECTURE.md)
- [フロントエンド設計書](./FRONTEND.md)
- [バックエンド設計書](./BACKEND.md)
- [統合・E2Eテスト](./INTEGRATION.md)
- [品質テスト](./QUALITY.md)
- [セキュリティ・脆弱性テスト](./SECURITY.md)
- [依存関係管理と自動更新](../implement/DEPENDENCIES.md)
