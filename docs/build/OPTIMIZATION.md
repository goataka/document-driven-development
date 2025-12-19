# ビルド最適化

本ドキュメントは、ビルドプロセスの最適化戦略を説明します。

**関連ドキュメント**: 
- [ビルド戦略概要](./README.md)
- [フロントエンドビルド](./FRONTEND.md)
- [バックエンドビルド](./BACKEND.md)

---

## フロントエンド最適化

### 1. コード分割

```typescript
// 動的インポートによるコード分割
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Attendance = lazy(() => import('./pages/Attendance'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/attendance" element={<Attendance />} />
      </Routes>
    </Suspense>
  );
}
```

### 2. 画像最適化

```typescript
// vite.config.ts
import imagemin from 'vite-plugin-imagemin';

export default defineConfig({
  plugins: [
    imagemin({
      gifsicle: { optimizationLevel: 7 },
      optipng: { optimizationLevel: 7 },
      mozjpeg: { quality: 80 },
      pngquant: { quality: [0.8, 0.9] },
      svgo: {
        plugins: [
          { name: 'removeViewBox', active: false },
          { name: 'removeEmptyAttrs', active: false }
        ]
      }
    })
  ]
});
```

### 3. キャッシュ戦略

- **ハッシュ化されたファイル名**: 長期キャッシュ可能
- **Vendor分割**: ライブラリの変更頻度を考慮
- **Service Worker**: オフライン対応（オプション）

---

## バックエンド最適化

### 1. 依存関係の最適化

```bash
# 本番用依存関係のみインストール
npm ci --production

# 不要なファイルを除外
# .dockerignore, .gitignore を活用
```

### 2. ビルド時の型チェック

```json
// package.json
{
  "scripts": {
    "prebuild": "npm run type-check",
    "build": "nest build",
    "type-check": "tsc --noEmit"
  }
}
```

---

## ベストプラクティス

1. **ビルド前のクリーン**: 前回のビルド成果物を削除
2. **型チェックの自動化**: ビルド前に必ず型チェックを実行
3. **環境変数の管理**: `.env.example`を提供し、機密情報は含めない
4. **ビルド成果物の検証**: ビルド後にサイズや構造を確認
5. **キャッシュの活用**: CI/CDでビルドキャッシュを利用
6. **並列ビルド**: フロントエンドとバックエンドを並列実行
7. **アーティファクトの保存**: CI/CDでビルド成果物を保存
8. **バージョン管理**: ビルド時にバージョン情報を埋め込む

---

## 関連ドキュメント

- [ビルド戦略概要](./README.md)
- [フロントエンドビルド](./FRONTEND.md)
- [バックエンドビルド](./BACKEND.md)
- [CI/CDビルド](./CI_CD.md)
- [トラブルシューティング](./TROUBLESHOOTING.md)

---

**最終更新日**: 2024年12月19日  
**バージョン**: 1.0.0
