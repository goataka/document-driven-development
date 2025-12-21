# スナップショットテスト

本ドキュメントは、UIコンポーネントのスナップショットテストの実装方法を説明します。

**関連ドキュメント**: 
- [テスト戦略概要](./)
- [システムアーキテクチャ設計書](../implement/)

## 目次

1. [スナップショットテスト](#スナップショットテスト)
2. [ベストプラクティス](#ベストプラクティス)

---

## スナップショットテスト

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

---

## ベストプラクティス

1. **小さな単位でテスト**: 大きなコンポーネントツリー全体ではなく、小さな単位でスナップショット
2. **頻繁に見直し**: スナップショットの更新時は差分を必ず確認
3. **意図的な変更のみ**: 意図しない変更がないか注意深くレビュー
4. **動的データの除外**: タイムスタンプやランダムIDなど動的データはモック化
5. **スナップショットは小さく保つ**: コンポーネント単位でスナップショット作成
6. **意図的な変更のみ更新**: 変更を確認してから更新、自動更新は慎重に
7. **重要なプロップをテスト**: 全バリエーションをカバー、エッジケースも含める

---
