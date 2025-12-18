# 品質テスト

本ドキュメントは、スナップショットテスト、アクセシビリティテスト、パフォーマンステストの実装方法を説明します。

**関連ドキュメント**: 
- [テスト戦略概要](./TESTING.md)
- [システムアーキテクチャ設計書](./ARCHITECTURE.md)

## 目次

1. [スナップショットテスト](#スナップショットテスト)
2. [アクセシビリティテスト](#アクセシビリティテスト)
3. [パフォーマンステスト](#パフォーマンステスト)
4. [ベストプラクティス](#ベストプラクティス)

---


スナップショットテストは、UIコンポーネントの予期しない変更を検出するために使用します。

### フロントエンド: Vitest + React

#### スナップショットテスト設定

```typescript
// vitest.config.ts（スナップショット設定を追加）
export default defineConfig({
  test: {
    // ... 他の設定
    snapshotSerializers: ['@testing-library/jest-dom/serializers'],
  }
});
```

#### コンポーネントスナップショットテスト例

```typescript
// src/components/Button/__tests__/Button.snapshot.test.tsx
import { render } from '@testing-library/react';
import { describe, it, expect } from 'vitest';
import { Button } from '../Button';

describe('Button スナップショットテスト', () => {
  it('デフォルトのボタンをスナップショット', () => {
    const { container } = render(<Button>クリック</Button>);
    expect(container.firstChild).toMatchSnapshot();
  });

  it('primary variantのボタンをスナップショット', () => {
    const { container } = render(
      <Button variant="primary">送信</Button>
    );
    expect(container.firstChild).toMatchSnapshot();
  });

  it('disabledボタンをスナップショット', () => {
    const { container } = render(
      <Button disabled>無効</Button>
    );
    expect(container.firstChild).toMatchSnapshot();
  });

  it('loadingボタンをスナップショット', () => {
    const { container } = render(
      <Button loading>読み込み中</Button>
    );
    expect(container.firstChild).toMatchSnapshot();
  });
});
```

#### スナップショット更新コマンド

```bash
# スナップショットを更新
npm test -- -u

# 特定のファイルのスナップショットを更新
npm test Button.snapshot.test.tsx -- -u
```

### ベストプラクティス

1. **小さな単位でテスト**: 大きなコンポーネントツリー全体ではなく、小さな単位でスナップショット
2. **頻繁に見直し**: スナップショットの更新時は差分を必ず確認
3. **意図的な変更のみ**: 意図しない変更がないか注意深くレビュー
4. **動的データの除外**: タイムスタンプやランダムIDなど動的データはモック化

---

## アクセシビリティテスト

アクセシビリティテストは、すべてのユーザーがアプリケーションを使用できることを保証します。

### ツール: jest-axe + axe-core

#### インストール

```bash
npm install --save-dev jest-axe axe-core @axe-core/playwright
```

#### jest-axe設定

```typescript
// src/test/setup.ts
import { toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);
```

#### コンポーネントアクセシビリティテスト例

```typescript
// src/components/LoginForm/__tests__/LoginForm.a11y.test.tsx
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';
import { describe, it, expect } from 'vitest';
import { LoginForm } from '../LoginForm';

expect.extend(toHaveNoViolations);

describe('LoginForm アクセシビリティテスト', () => {
  it('WCAG 2.1 Level AA基準に準拠すること', async () => {
    const { container } = render(<LoginForm />);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });

  it('フォーム要素に適切なラベルがあること', async () => {
    const { container, getByLabelText } = render(<LoginForm />);
    
    // ラベルで要素を取得できることを確認
    expect(getByLabelText('メールアドレス')).toBeInTheDocument();
    expect(getByLabelText('パスワード')).toBeInTheDocument();
    
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });

  it('エラーメッセージが適切にアナウンスされること', async () => {
    const { container, getByRole } = render(
      <LoginForm error="認証に失敗しました" />
    );
    
    // aria-liveリージョンが存在することを確認
    const alert = getByRole('alert');
    expect(alert).toHaveTextContent('認証に失敗しました');
    
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });
});
```

### E2E アクセシビリティテスト (Playwright)

```typescript
// e2e/tests/accessibility.spec.ts
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test.describe('アクセシビリティテスト', () => {
  test('トップページがアクセシビリティ基準を満たすこと', async ({ page }) => {
    await page.goto('/');
    
    const accessibilityScanResults = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa'])
      .analyze();
    
    expect(accessibilityScanResults.violations).toEqual([]);
  });

  test('ログインページがキーボード操作可能なこと', async ({ page }) => {
    await page.goto('/login');
    
    // Tab キーで全てのインタラクティブ要素にアクセス可能
    await page.keyboard.press('Tab'); // メールアドレスフィールド
    await expect(page.locator('[name="email"]')).toBeFocused();
    
    await page.keyboard.press('Tab'); // パスワードフィールド
    await expect(page.locator('[name="password"]')).toBeFocused();
    
    await page.keyboard.press('Tab'); // ログインボタン
    await expect(page.locator('button[type="submit"]')).toBeFocused();
    
    const accessibilityScanResults = await new AxeBuilder({ page })
      .analyze();
    
    expect(accessibilityScanResults.violations).toEqual([]);
  });

  test('フォームバリデーションエラーが適切にアナウンスされること', async ({ page }) => {
    await page.goto('/login');
    
    // 空でログインを試行
    await page.click('button[type="submit"]');
    
    // エラーメッセージのaria-live属性を確認
    const errorMessage = page.locator('[role="alert"]');
    await expect(errorMessage).toBeVisible();
    
    const accessibilityScanResults = await new AxeBuilder({ page })
      .analyze();
    
    expect(accessibilityScanResults.violations).toEqual([]);
  });
});
```

### アクセシビリティチェックリスト

- ✅ すべてのフォーム要素に適切な`<label>`がある
- ✅ 画像に代替テキスト（`alt`属性）がある
- ✅ 色だけで情報を伝えていない
- ✅ キーボードだけで全操作が可能
- ✅ フォーカスインジケーターが明確
- ✅ 適切なARIA属性（`role`, `aria-label`, `aria-live`など）
- ✅ 十分なコントラスト比（4.5:1以上）
- ✅ スクリーンリーダーで適切に読み上げられる

---

## パフォーマンステスト

パフォーマンステストは、アプリケーションの応答速度と効率性を測定します。

### Lighthouse CI

#### インストールと設定

```bash
npm install --save-dev @lhci/cli
```

```javascript
// lighthouserc.js
module.exports = {
  ci: {
    collect: {
      url: ['http://localhost:3000/', 'http://localhost:3000/login'],
      numberOfRuns: 3,
      settings: {
        preset: 'desktop',
      },
    },
    assert: {
      preset: 'lighthouse:recommended',
      assertions: {
        'categories:performance': ['error', { minScore: 0.9 }],
        'categories:accessibility': ['error', { minScore: 0.9 }],
        'categories:best-practices': ['error', { minScore: 0.9 }],
        'categories:seo': ['error', { minScore: 0.9 }],
        // Core Web Vitals
        'first-contentful-paint': ['error', { maxNumericValue: 2000 }],
        'largest-contentful-paint': ['error', { maxNumericValue: 2500 }],
        'cumulative-layout-shift': ['error', { maxNumericValue: 0.1 }],
        'total-blocking-time': ['error', { maxNumericValue: 300 }],
      },
    },
    upload: {
      target: 'temporary-public-storage',
    },
  },
};
```

#### Lighthouse CI コマンド

```bash
# Lighthouse CI 実行
npm run lhci:collect
npm run lhci:assert

# package.json に追加
"scripts": {
  "lhci:collect": "lhci collect",
  "lhci:assert": "lhci assert",
  "lhci:upload": "lhci upload"
}
```

### Playwright パフォーマンステスト

```typescript
// e2e/tests/performance.spec.ts
import { test, expect } from '@playwright/test';

test.describe('パフォーマンステスト', () => {
  test('トップページが3秒以内に読み込まれること', async ({ page }) => {
    const startTime = Date.now();
    
    await page.goto('/', { waitUntil: 'networkidle' });
    
    const loadTime = Date.now() - startTime;

---

## ベストプラクティス

### スナップショットテスト

1. **スナップショットは小さく保つ**
   - コンポーネント単位でスナップショット作成
   - 大きすぎるスナップショットは避ける

2. **意図的な変更のみ更新**
   - 変更を確認してから更新
   - 自動更新は慎重に

3. **重要なプロップをテスト**
   - 全バリエーションをカバー
   - エッジケースも含める

### アクセシビリティテスト

1. **開発中から確認**
   - コンポーネント作成時にチェック
   - PR前に必ず実行

2. **キーボード操作を確認**
   - Tab キーでの遷移
   - Enter/Space での操作

3. **スクリーンリーダーで確認**
   - 実際のツールでテスト
   - ラベルが適切か確認

### パフォーマンステスト

1. **定期的に測定**
   - CI/CDで自動実行
   - 劣化を早期検知

2. **実際のデータ量で測定**
   - 本番相当のデータ
   - 負荷がかかる状況

3. **ボトルネックを特定**
   - プロファイリングツール活用
   - データベースクエリ最適化

---

## 関連ドキュメント

- [テスト戦略概要](./TESTING.md)
- [統合・E2Eテスト](./TESTING_INTEGRATION.md)
- [セキュリティ・脆弱性テスト](./TESTING_SECURITY.md)
- [依存関係管理と自動更新](./TESTING_DEPENDENCIES.md)
