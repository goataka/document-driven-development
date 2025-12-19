# セキュリティ

本ドキュメントは、システムのセキュリティ設計と実装の詳細を説明します。

**関連ドキュメント**: [システムアーキテクチャ設計書](../implement/)

## 目次

1. [認証・認可](#認証認可)
2. [セキュリティ対策](#セキュリティ対策)
3. [実装例](#実装例)

---

## 認証・認可

### 1. JWT（JSON Web Token）認証

- ログイン時にJWTトークンを発行
- トークンの有効期限: 7日間
- リフレッシュトークンの実装を推奨（将来的な拡張）

### 2. パスワード管理

- **bcrypt**を使用したハッシュ化（salt rounds: 10以上）
- パスワードポリシー:
  - 最小8文字
  - 英大文字・小文字・数字を含む
  - 特殊文字を推奨

### 3. 認可制御

- ユーザーは自分自身のデータのみアクセス可能
- 会社コードによる組織分離

---

## セキュリティ対策

### 1. CORS設定

環境変数で許可するオリジンを設定し、本番環境では厳格に管理

#### 実装例

```typescript
// src/main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.enableCors({
    origin: process.env.FRONTEND_URL,
    credentials: true
  });
  
  await app.listen(3000);
}
bootstrap();
```

---

### 2. ヘルメット（Helmet）

セキュリティヘッダーの設定によるXSS、クリックジャッキング等の対策

#### 実装例

```typescript
import helmet from 'helmet';
app.use(helmet());
```

---

### 3. レート制限

DDoS攻撃やブルートフォース攻撃の防止

#### 実装例

```typescript
import rateLimit from 'express-rate-limit';

app.use(
  rateLimit({
    windowMs: 15 * 60 * 1000, // 15分
    max: 100 // 最大100リクエスト
  })
);
```

---

### 4. バリデーション

- すべてのユーザー入力をバリデーション
- **class-validator**を使用したDTOレベルのバリデーション
- SQLインジェクション対策（ORMの使用）
- XSS対策（入力のサニタイゼーション）

---

### 5. HTTPS強制

- 本番環境では必ずHTTPSを使用
- HTTP Strict Transport Security (HSTS)ヘッダーの設定

---

## 実装例

詳細な実装例については、各セクションに記載されています。

**関連ドキュメント**:
- [システムアーキテクチャ設計書](../implement/)
- [バックエンド設計](./BACKEND.md)
- [API設計](./API.md)

---

**最終更新日**: 2024年12月17日  
**バージョン**: 1.0.0
