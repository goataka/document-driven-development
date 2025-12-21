# ADR-012: API Gatewayの統合

## ステータス

提案中（Proposed）

## 背景

Lambda関数へのHTTPアクセスを管理するため、API Gatewayの導入を検討しています。

### API Gateway とは

AWSが提供するフルマネージドなAPIサービスで、RESTful API、HTTP API、WebSocket APIをサポートします。Lambda関数への入り口として機能します。

### API Gateway の種類

1. **HTTP API**: シンプルで低コスト、基本的なAPI機能
2. **REST API**: 高機能、APIキー、リクエスト/レスポンス変換、キャッシュ等
3. **WebSocket API**: リアルタイム双方向通信

## 決定事項

**オプション**: APIエンドポイント管理が必要な場合にAPI Gatewayを導入する。初期実装では検討事項とする。

### API Gateway を導入すべきケース

1. **APIエンドポイント管理**: 複数のLambda関数を統合管理
2. **認証・認可**: Cognito、JWTトークン検証
3. **レート制限**: APIコール数の制限
4. **API仕様管理**: OpenAPI仕様の自動生成
5. **キャッシュ**: レスポンスキャッシュによるパフォーマンス向上
6. **カスタムドメイン**: 独自ドメインの使用

### API Gateway なしで十分なケース

1. **シンプルな構成**: Lambda単体で十分
2. **Function URL使用**: Lambda Function URLで直接公開
3. **コスト重視**: API Gatewayのコストを削減したい
4. **低トラフィック**: APIコール数が少ない

## 結果

### メリット（API Gateway 導入）

- **統合管理**: 複数のLambda関数を一元管理
- **認証・認可**: Cognito、カスタムオーソライザー
- **レート制限**: スロットリング、クォータ管理
- **CORS設定**: クロスオリジン対応
- **モニタリング**: CloudWatch統合
- **API仕様**: OpenAPI/Swagger自動生成
- **バージョニング**: ステージ管理（dev、staging、prod）

### デメリット

- **追加コスト**: リクエストごとの課金
- **レイテンシ**: 若干の遅延（1-2ms）
- **設定の複雑さ**: IAM、リソースポリシー
- **学習コスト**: API Gatewayの理解が必要

### コスト比較

#### HTTP API（推奨）
- **リクエスト料金**: $1.00 per million requests
- **月間100万リクエスト**: $1.00
- **月間1000万リクエスト**: $10.00

#### REST API
- **リクエスト料金**: $3.50 per million requests
- **月間100万リクエスト**: $3.50
- **月間1000万リクエスト**: $35.00

#### Lambda Function URL（無料）
- **リクエスト料金**: $0
- **制限**: 認証・認可機能が限定的

## 実装例

### 1. Lambda Function URL（API Gateway なし）

```typescript
// Lambda関数にFunction URLを設定
resource "aws_lambda_function_url" "app" {
  function_name      = aws_lambda_function.app.function_name
  authorization_type = "AWS_IAM" // または "NONE"

  cors {
    allow_origins = ["https://example.com"]
    allow_methods = ["GET", "POST"]
    allow_headers = ["Content-Type", "Authorization"]
    max_age       = 3600
  }
}

output "function_url" {
  value = aws_lambda_function_url.app.function_url
  // https://xxx.lambda-url.ap-northeast-1.on.aws/
}
```

### 2. HTTP API（推奨）

```typescript
// HTTP API作成
resource "aws_apigatewayv2_api" "app" {
  name          = "attendance-api"
  protocol_type = "HTTP"

  cors_configuration {
    allow_origins = ["https://example.com"]
    allow_methods = ["GET", "POST", "PUT", "DELETE"]
    allow_headers = ["Content-Type", "Authorization"]
    max_age       = 3600
  }
}

// Lambda統合
resource "aws_apigatewayv2_integration" "lambda" {
  api_id           = aws_apigatewayv2_api.app.id
  integration_type = "AWS_PROXY"
  integration_uri  = aws_lambda_function.app.invoke_arn
}

// ルート設定
resource "aws_apigatewayv2_route" "users" {
  api_id    = aws_apigatewayv2_api.app.id
  route_key = "GET /users/{userId}"
  target    = "integrations/${aws_apigatewayv2_integration.lambda.id}"

  authorization_type = "JWT"
  authorizer_id      = aws_apigatewayv2_authorizer.jwt.id
}

// ステージ
resource "aws_apigatewayv2_stage" "prod" {
  api_id      = aws_apigatewayv2_api.app.id
  name        = "prod"
  auto_deploy = true

  access_log_settings {
    destination_arn = aws_cloudwatch_log_group.api_logs.arn
    format         = "$context.requestId"
  }
}
```

### 3. NestJS with API Gateway

```typescript
// Lambda Handler
import { NestFactory } from '@nestjs/core';
import { ExpressAdapter } from '@nestjs/platform-express';
import serverlessExpress from '@vendia/serverless-express';
import { AppModule } from './app.module';
import express from 'express';

let cachedServer;

async function bootstrap() {
  if (!cachedServer) {
    const expressApp = express();
    const nestApp = await NestFactory.create(
      AppModule,
      new ExpressAdapter(expressApp),
    );
    
    nestApp.enableCors({
      origin: process.env.FRONTEND_URL,
      credentials: true,
    });

    await nestApp.init();
    cachedServer = serverlessExpress({ app: expressApp });
  }
  return cachedServer;
}

export const handler = async (event, context) => {
  const server = await bootstrap();
  return server(event, context);
};
```

## アーキテクチャ比較

### Lambda Function URL

```
クライアント
    ↓ HTTPS
Lambda Function URL
    ↓
Lambda関数
```

**メリット**: シンプル、低コスト  
**デメリット**: 機能が限定的

### HTTP API

```
クライアント
    ↓ HTTPS
API Gateway (HTTP API)
    ↓
Lambda関数
```

**メリット**: 中程度の機能、低コスト  
**デメリット**: REST APIより機能が少ない

### REST API

```
クライアント
    ↓ HTTPS
API Gateway (REST API)
    ↓ (キャッシュ、変換)
Lambda関数
```

**メリット**: 高機能  
**デメリット**: 高コスト

## 認証・認可の実装

### Lambda Function URL + JWT検証

```typescript
import { verify } from 'jsonwebtoken';

export const handler = async (event) => {
  try {
    const token = event.headers.authorization?.replace('Bearer ', '');
    const decoded = verify(token, process.env.JWT_SECRET);
    
    // ユーザー情報をコンテキストに追加
    event.requestContext.authorizer = decoded;
    
    // ビジネスロジック実行
    return {
      statusCode: 200,
      body: JSON.stringify({ message: 'Success' }),
    };
  } catch (error) {
    return {
      statusCode: 401,
      body: JSON.stringify({ error: 'Unauthorized' }),
    };
  }
};
```

### HTTP API + JWT Authorizer

```typescript
// API Gatewayで自動的にJWT検証
resource "aws_apigatewayv2_authorizer" "jwt" {
  api_id           = aws_apigatewayv2_api.app.id
  authorizer_type  = "JWT"
  identity_sources = ["$request.header.Authorization"]
  name             = "jwt-authorizer"

  jwt_configuration {
    audience = ["api"]
    issuer   = "https://cognito-idp.ap-northeast-1.amazonaws.com/..."
  }
}

// Lambda関数では認証済みユーザー情報を取得
export const handler = async (event) => {
  const userId = event.requestContext.authorizer.claims.sub;
  // ビジネスロジック実行
};
```

## パフォーマンス比較

| 構成 | レイテンシ | スループット | コスト（100万リクエスト） |
|------|-----------|-------------|------------------------|
| Function URL | 10ms | 高 | $0 |
| HTTP API | 11-12ms | 高 | $1.00 |
| REST API | 12-15ms | 中 | $3.50 |

## 推奨構成

### 開発環境
- **Lambda Function URL**: シンプルで十分
- **認証**: JWT検証をLambda内で実装

### ステージング・本番環境
- **HTTP API**: コストと機能のバランスが良い
- **認証**: JWT Authorizer

### エンタープライズ
- **REST API**: 高度な機能が必要な場合
- **キャッシュ**: レスポンスキャッシュ有効化

## 段階的導入戦略

### Phase 1（現在）: Lambda Function URL
- シンプルな構成
- 低コスト

### Phase 2（中期）: HTTP API導入
- 複数エンドポイント管理
- JWT Authorizer

### Phase 3（長期）: REST API（必要に応じて）
- 高度な機能が必要な場合
- APIキャッシュ、変換機能

## 学習リソース

- [API Gateway Overview](https://aws.amazon.com/api-gateway/)
- [HTTP APIs vs REST APIs](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-vs-rest.html)
- [Lambda Function URLs](https://docs.aws.amazon.com/lambda/latest/dg/lambda-urls.html)

## 関連ドキュメント

- [バックエンドデプロイ方針](../../docs/deploy/BACKEND.md)
- [API設計](../../docs/implement/API.md)
- [セキュリティ](../../docs/implement/SECURITY.md)

## 更新履歴

- 2024-12-21: 初版作成
