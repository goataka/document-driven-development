# ADR-004: Redisキャッシュの導入

## ステータス

提案中（Proposed）

## 背景

アプリケーションのパフォーマンス向上とスケーラビリティを確保するため、キャッシュ層の導入を検討しています。

主な用途：
- セッション管理
- データベースクエリ結果のキャッシュ
- レート制限の実装
- 一時データの保存

## 決定事項

**オプション**: Redis をセッション管理とキャッシュ用途で導入可能とする。ただし、初期実装では必須とせず、パフォーマンス要件に応じて段階的に導入する。

### Redis を導入するメリット

1. **パフォーマンス向上**: インメモリストアによる高速アクセス
2. **セッション管理**: 複数サーバー間でのセッション共有
3. **スケーラビリティ**: 水平スケーリングへの対応
4. **データ構造**: リスト、セット、ソート済みセットなど豊富なデータ構造
5. **TTL**: 自動的な有効期限管理

### Redis なしで運用する場合

1. **JWTトークンベース認証**: ステートレスな認証
2. **DynamoDB TTL**: データの自動削除
3. **アプリケーションレベルキャッシュ**: メモリ内キャッシュ（単一インスタンス）

## 結果

### メリット（Redis 導入の場合）

- セッション管理の柔軟性
- データベース負荷の軽減
- レスポンスタイムの改善
- 複数インスタンス間でのデータ共有

### デメリット

- インフラストラクチャの複雑化
- 運用コストの増加（ElastiCache等）
- データ整合性の考慮が必要
- 追加の監視・メンテナンス

### 導入判断基準

以下の場合に Redis の導入を検討すべき：

1. **高トラフィック**: 秒間100リクエスト以上
2. **セッション管理**: 複数サーバー間でのセッション共有が必要
3. **複雑なレート制限**: IPベース、ユーザーベースのレート制限
4. **リアルタイム機能**: Pub/Subを使用したリアルタイム通知

### 段階的導入戦略

1. **Phase 1（初期）**: JWTベース認証のみ
2. **Phase 2（中期）**: セッション管理のために Redis 導入
3. **Phase 3（長期）**: クエリキャッシュ、レート制限の実装

## 実装例

### Redis を使用したセッション管理

```typescript
import { Injectable } from '@nestjs/common';
import { Redis } from 'ioredis';

@Injectable()
export class SessionService {
  private readonly redis: Redis;

  constructor() {
    this.redis = new Redis({
      host: process.env.REDIS_HOST,
      port: parseInt(process.env.REDIS_PORT),
      password: process.env.REDIS_PASSWORD,
    });
  }

  async setSession(sessionId: string, data: any, ttl: number = 3600): Promise<void> {
    await this.redis.setex(
      `session:${sessionId}`,
      ttl,
      JSON.stringify(data)
    );
  }

  async getSession(sessionId: string): Promise<any> {
    const data = await this.redis.get(`session:${sessionId}`);
    return data ? JSON.parse(data) : null;
  }

  async deleteSession(sessionId: string): Promise<void> {
    await this.redis.del(`session:${sessionId}`);
  }
}
```

### JWTベース認証（Redis なし）

```typescript
import { Injectable } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';

@Injectable()
export class AuthService {
  constructor(private readonly jwtService: JwtService) {}

  async login(user: User): Promise<{ accessToken: string }> {
    const payload = { sub: user.id, email: user.email };
    return {
      accessToken: this.jwtService.sign(payload),
    };
  }

  async validateToken(token: string): Promise<any> {
    return this.jwtService.verify(token);
  }
}
```

## 環境別構成

### ローカル環境
- Docker Compose で Redis コンテナを起動
- ポート: 6379（デフォルト）

### CI環境
- Redis コンテナをサービスとして起動
- テスト完了後に自動削除

### 本番環境
- AWS ElastiCache for Redis
- マルチAZ構成
- 自動フェイルオーバー

## コスト見積もり

### Redis を導入する場合
- AWS ElastiCache (cache.t4g.micro): 月額 $12-15
- データ転送料: 月額 $1-3
- **合計**: 月額 $13-18

### Redis なしの場合
- コスト: $0
- ただし、DynamoDB読み取り/書き込みコストが増加する可能性

## 関連ドキュメント

- [システムアーキテクチャ設計書](../../docs/implement/README.md)
- [バックエンド設計](../../docs/implement/BACKEND.md)
- [バックエンドデプロイ方針](../../docs/deploy/BACKEND.md)

## 更新履歴

- 2024-12-21: 初版作成
