# フロントエンドデプロイ

本ドキュメントは、フロントエンドのデプロイ方法を説明します。

**関連ドキュメント**: 
- [デプロイ戦略概要](./README.md)
- [バックエンドデプロイ](./BACKEND.md)

## Vercel デプロイ

```bash
npm install -g vercel
vercel login
cd frontend
vercel link
vercel --prod
```

## Netlify デプロイ

```toml
[build]
  command = "npm run build"
  publish = "dist"
```

---

**最終更新日**: 2024年12月19日  
**バージョン**: 1.0.0
