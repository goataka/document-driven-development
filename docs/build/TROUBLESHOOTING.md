# トラブルシューティング

本ドキュメントは、ビルド時の一般的な問題と解決方法を説明します。

**関連ドキュメント**: 
- [ビルド戦略概要](./README.md)
- [フロントエンドビルド](./FRONTEND.md)
- [バックエンドビルド](./BACKEND.md)

---

## よくある問題と解決方法

### 1. メモリ不足エラー

```bash
# Node.jsのメモリ上限を増やす
NODE_OPTIONS="--max-old-space-size=4096" npm run build
```

### 2. 型エラー

```bash
# 型定義を再インストール
rm -rf node_modules package-lock.json
npm install

# 型チェックのみ実行
npm run type-check
```

### 3. ビルド時間が長い

```typescript
// vite.config.ts - 開発用設定を本番から分離
export default defineConfig(({ mode }) => ({
  build: {
    minify: mode === 'production' ? 'terser' : false,
    sourcemap: mode === 'production' ? true : 'inline',
  }
}));
```

### 4. 依存関係の競合

```bash
# package-lock.jsonを削除して再インストール
rm package-lock.json
npm install

# または特定のバージョンを指定
npm install package-name@specific-version
```

---

## ビルドログの確認

```bash
# 詳細ログ付きビルド
npm run build -- --verbose

# ビルドサイズ分析
npm run build -- --report
```

---

## 関連ドキュメント

- [ビルド戦略概要](./README.md)
- [フロントエンドビルド](./FRONTEND.md)
- [バックエンドビルド](./BACKEND.md)
- [ビルド最適化](./OPTIMIZATION.md)
- [CI/CDビルド](./CI_CD.md)

---

**最終更新日**: 2024年12月19日  
**バージョン**: 1.0.0
