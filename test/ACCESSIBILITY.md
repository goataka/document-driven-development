# アクセシビリティテスト

本ドキュメントは、アクセシビリティテスト(WCAG 2.1 Level AA準拠)の実装方法を説明します。

**関連ドキュメント**: 
- [テスト戦略概要](./TESTING.md)
- [システムアーキテクチャ設計書](../spec/ARCHITECTURE.md)

## 目次

1. [アクセシビリティテスト](#アクセシビリティテスト)
2. [ベストプラクティス](#ベストプラクティス)

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

## ベストプラクティス

1. **開発中から確認**
   - コンポーネント作成時にチェック
   - PR前に必ず実行

2. **キーボード操作を確認**
   - Tab キーでの遷移
   - Enter/Space での操作

3. **スクリーンリーダーで確認**
   - 実際のツールでテスト
   - ラベルが適切か確認

---

## 関連ドキュメント

- [テスト戦略概要](./TESTING.md)
- [統合・E2Eテスト](./INTEGRATION.md)
- [スナップショットテスト](./SNAPSHOT.md)
- [パフォーマンステスト](./PERFORMANCE.md)
