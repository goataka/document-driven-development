# フロントエンドビルド

本ドキュメントは、フロントエンド（React + Vite）のビルドプロセスを説明します。

**関連ドキュメント**: 
- [ビルド戦略概要](./README.md)
- [バックエンドビルド](./BACKEND.md)
- [ビルド最適化](./OPTIMIZATION.md)

---

## ビルドプロセス

```bash
# ディレクトリ移動
cd frontend

# 依存関係インストール
npm ci

# 型チェック
npm run type-check

# ビルド実行
npm run build

# ビルド結果は dist/ に出力される
```

---

## Vite設定

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  build: {
    outDir: 'dist',
    sourcemap: true,
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true, // 本番環境ではconsole.logを削除
      }
    },
    rollupOptions: {
      output: {
        manualChunks: {
          'react-vendor': ['react', 'react-dom', 'react-router-dom'],
          'mui': ['@mui/material', '@mui/icons-material'],
        }
      }
    },
    chunkSizeWarningLimit: 1000,
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    }
  }
});
```

---

## ビルド成果物

```
frontend/dist/
├── index.html              # エントリーポイント
├── assets/
│   ├── index-[hash].js     # メインJavaScript
│   ├── react-vendor-[hash].js  # Reactライブラリ
│   ├── mui-[hash].js       # Material-UIライブラリ
│   └── index-[hash].css    # スタイルシート
└── favicon.ico
```

---

## 環境変数

### .env.production

```env
VITE_API_URL=https://api.example.com
VITE_APP_NAME=勤怠管理システム
VITE_APP_VERSION=1.0.0
```

---

## 関連ドキュメント

- [ビルド戦略概要](./README.md)
- [バックエンドビルド](./BACKEND.md)
- [ビルド最適化](./OPTIMIZATION.md)
- [CI/CDビルド](./CI_CD.md)
- [トラブルシューティング](./TROUBLESHOOTING.md)

---

**最終更新日**: 2024年12月19日  
**バージョン**: 1.0.0
