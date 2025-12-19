# バックエンドデプロイ

本ドキュメントは、バックエンドのデプロイ方法を説明します。

**関連ドキュメント**: 
- [デプロイ戦略概要](./README.md)
- [フロントエンドデプロイ](./FRONTEND.md)

## Docker コンテナ化

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/main.js"]
```

---

**最終更新日**: 2024年12月19日  
**バージョン**: 1.0.0
