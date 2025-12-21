# ADR-002: UIライブラリの選定

## ステータス

提案中（Proposed）

## 背景

フロントエンドアプリケーションの開発効率を高め、一貫性のあるUIを実現するため、UIコンポーネントライブラリの採用を検討しています。

現在、以下の候補を検討しています：

- **Material-UI (MUI)**: Googleのマテリアルデザインベース
- **Ant Design**: エンタープライズ向けUIライブラリ
- **Chakra UI**: アクセシビリティ重視のモダンなUIライブラリ
- **shadcn/ui**: Radix UIベースのカスタマイズ可能なコンポーネント

## 決定事項

**推奨**: Material-UI (MUI) を第一選択とし、デザイン要件により他ライブラリも検討可能とする。

### Material-UI を推奨する理由

1. **成熟度**: 長い開発履歴と大規模なコミュニティ
2. **コンポーネントの豊富さ**: 包括的なコンポーネントセット
3. **カスタマイズ性**: テーマシステムによる柔軟なカスタマイズ
4. **TypeScript サポート**: 優れた型定義
5. **アクセシビリティ**: WAI-ARIA準拠
6. **ドキュメント**: 充実したドキュメントと例

### 他のライブラリを検討すべきケース

#### Ant Design
- 中国企業向けのデザイン要件がある場合
- エンタープライズ向けの複雑なテーブル・フォームが多い場合

#### Chakra UI
- よりモダンなAPI（Styled System）を求める場合
- アクセシビリティを最優先する場合
- ダークモード対応が重要な場合

#### shadcn/ui
- 完全なカスタマイズ性を求める場合
- コンポーネントをプロジェクトに直接統合したい場合
- バンドルサイズを最小限に抑えたい場合

## 結果

### メリット（Material-UI 採用の場合）

- 開発速度の向上（既製コンポーネントの活用）
- 一貫性のあるデザインシステム
- デザイナーとの連携が容易（マテリアルデザインガイドライン）
- 豊富なサードパーティ統合（Data Grid, Date Picker等）

### デメリット

- バンドルサイズが大きい（Tree Shakingで軽減可能）
- マテリアルデザインに縛られる可能性
- 過度なカスタマイズは複雑になる

### パフォーマンス対策

- Tree Shaking の有効化
- 必要なコンポーネントのみをインポート
- Code Splitting の活用

## 実装例

### Material-UI の基本的な使用例

```typescript
import { Button, TextField, Box } from '@mui/material';
import { createTheme, ThemeProvider } from '@mui/material/styles';

const theme = createTheme({
  palette: {
    primary: {
      main: '#1976d2',
    },
  },
});

function App() {
  return (
    <ThemeProvider theme={theme}>
      <Box sx={{ p: 2 }}>
        <TextField label="Email" variant="outlined" fullWidth sx={{ mb: 2 }} />
        <Button variant="contained" color="primary">
          Submit
        </Button>
      </Box>
    </ThemeProvider>
  );
}
```

## 関連ドキュメント

- [システムアーキテクチャ設計書](../../docs/implement/README.md)
- [フロントエンド設計](../../docs/implement/FRONTEND.md)

## 更新履歴

- 2024-12-21: 初版作成
