# バックエンドビルド

本ドキュメントは、バックエンド（NestJS）のビルドプロセスを説明します。

**関連ドキュメント**: 
- [ビルド戦略概要](./README.md)
- [フロントエンドビルド](./FRONTEND.md)
- [ビルド最適化](./OPTIMIZATION.md)

---

## ビルドプロセス

```bash
# ディレクトリ移動
cd backend

# 依存関係インストール
npm ci

# 型チェック
npm run type-check

# ビルド実行
npm run build

# ビルド結果は dist/ に出力される
```

---

## NestJS設定

```json
// nest-cli.json
{
  "collection": "@nestjs/schematics",
  "sourceRoot": "src",
  "compilerOptions": {
    "deleteOutDir": true,
    "assets": ["**/*.proto"],
    "watchAssets": true
  }
}
```

---

## TypeScript設定

```json
// tsconfig.build.json
{
  "extends": "./tsconfig.json",
  "exclude": [
    "node_modules",
    "test",
    "dist",
    "**/*spec.ts",
    "**/*test.ts"
  ]
}
```

---

## ビルド成果物

```
backend/dist/
├── main.js                 # エントリーポイント
├── app.module.js
├── config/
├── common/
├── modules/
│   ├── auth/
│   ├── users/
│   └── attendance/
└── database/
```

---

## 環境変数

### .env.production

```env
NODE_ENV=production
PORT=3000
DATABASE_URL=postgresql://user:password@localhost:5432/attendance_db
JWT_SECRET=your-production-secret-key
JWT_EXPIRATION=7d
CORS_ORIGIN=https://example.com
```

---

## 関連ドキュメント

- [ビルド戦略概要](./README.md)
- [フロントエンドビルド](./FRONTEND.md)
- [ビルド最適化](./OPTIMIZATION.md)
- [CI/CDビルド](./CI_CD.md)
- [トラブルシューティング](./TROUBLESHOOTING.md)

---

**最終更新日**: 2024年12月19日  
**バージョン**: 1.0.0
