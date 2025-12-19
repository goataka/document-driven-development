# コンポーネントテスト (Storybook)

本ドキュメントは、Storybookを使ったコンポーネントテストの実装方法を説明します。

**関連ドキュメント**: 
- [テスト戦略概要](./OVERVIEW.md)
- [単体テスト](./UNIT.md)
- [システムアーキテクチャ設計書](../implement/ARCHITECTURE.md)

---


## コンポーネントテスト (Storybook)

**目的**:
- コンポーネントの視覚的な確認
- 様々な状態（props）での動作確認
- UIカタログの作成
- デザインシステムの文書化

### Storybook設定

```typescript
// .storybook/main.ts
import type { StorybookConfig } from '@storybook/react-vite';

const config: StorybookConfig = {
  stories: ['../src/**/*.stories.@(js|jsx|ts|tsx)'],
  addons: [
    '@storybook/addon-links',
    '@storybook/addon-essentials',
    '@storybook/addon-interactions',
    '@storybook/addon-a11y',
  ],
  framework: {
    name: '@storybook/react-vite',
    options: {},
  },
  docs: {
    autodocs: 'tag',
  },
};

export default config;
```

### Story例

```typescript
// src/components/Button/Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';

const meta: Meta<typeof Button> = {
  title: 'Components/Button',
  component: Button,
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary'],
    },
    disabled: {
      control: 'boolean',
    },
  },
};

export default meta;
type Story = StoryObj<typeof Button>;

export const Primary: Story = {
  args: {
    label: '出勤打刻',
    variant: 'primary',
    onClick: () => alert('出勤打刻しました'),
  },
};

export const Secondary: Story = {
  args: {
    label: 'キャンセル',
    variant: 'secondary',
    onClick: () => alert('キャンセルしました'),
  },
};

export const Disabled: Story = {
  args: {
    label: '無効ボタン',
    variant: 'primary',
    disabled: true,
    onClick: () => {},
  },
};
```

### Interactionテスト

```typescript
// src/components/LoginForm/LoginForm.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { within, userEvent, expect } from '@storybook/test';
import { LoginForm } from './LoginForm';

const meta: Meta<typeof LoginForm> = {
  title: 'Forms/LoginForm',
  component: LoginForm,
};

export default meta;
type Story = StoryObj<typeof LoginForm>;

export const FilledForm: Story = {
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);

    // メールアドレス入力
    await userEvent.type(
      canvas.getByLabelText('メールアドレス'),
      'user@example.com'
    );

    // パスワード入力
    await userEvent.type(
      canvas.getByLabelText('パスワード'),
      'password123'
    );

    // ボタンが有効になることを確認
    const submitButton = canvas.getByRole('button', { name: 'ログイン' });
    await expect(submitButton).not.toBeDisabled();
  },
};
```

**実行コマンド**:

```bash
# Storybookの起動
npm run storybook

# ビルド
npm run build-storybook
```

---



---

## テストカバレッジ目標

| テストタイプ | 目標カバレッジ | 対象 |
|------------|-------------|------|
| 単体テスト | 80%以上 | ビジネスロジック、ユーティリティ関数 |
| スナップショットテスト | 全UIコンポーネント | UIコンポーネントの視覚的回帰 |
| 統合テスト | 主要APIエンドポイント全て | API層 |
| E2Eテスト | 主要ユーザーフロー全て | システム全体 |
| アクセシビリティテスト | 全ページ・コンポーネント | WCAG 2.1 Level AA準拠 |
