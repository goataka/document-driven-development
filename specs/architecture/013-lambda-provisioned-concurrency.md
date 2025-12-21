# ADR-013: Lambda Provisioned Concurrencyの導入

## ステータス

提案中（Proposed）

## 背景

Lambda関数のコールドスタート問題を解決するため、Provisioned Concurrency の導入を検討しています。

### コールドスタートとは

Lambda関数が初めて呼び出されるとき、または一定期間未使用後に呼び出されるとき、実行環境の初期化に時間がかかる現象です。

### コールドスタートの影響

- **初回実行時間**: 1-5秒（Node.js）、3-10秒（Java）
- **ユーザー体験**: レスポンス遅延
- **タイムアウト**: 長時間の初期化でタイムアウト

## 決定事項

**検討中**: 本番環境のみ、必要に応じて Provisioned Concurrency を導入する。初期実装では不要。

### Provisioned Concurrency を導入すべきケース

1. **レイテンシ要件**: 常に100ms以下のレスポンスが必要
2. **ピークトラフィック**: 急激なトラフィック増加に対応
3. **ユーザー体験重視**: コールドスタートを許容できない
4. **同期API**: リアルタイムレスポンスが必要
5. **本番環境**: SLA要件がある

### Provisioned Concurrency なしで十分なケース

1. **非同期処理**: キュー処理、バッチ処理
2. **低トラフィック**: コールドスタートが稀
3. **コスト重視**: 追加コストを避けたい
4. **開発環境**: レイテンシ要件が低い

## 結果

### メリット（Provisioned Concurrency 導入）

- **予測可能なレイテンシ**: コールドスタート排除
- **即座のスケーリング**: 事前にウォームアップ
- **ユーザー体験向上**: 常に高速レスポンス
- **SLA保証**: パフォーマンス要件を満たす

### デメリット

- **高コスト**: 常時実行環境を維持（$0.015/GB-hour）
- **過剰プロビジョニング**: 未使用時もコスト発生
- **設定の複雑さ**: 適切な同時実行数の決定が困難

### コスト比較

#### オンデマンド Lambda（Provisioned Concurrency なし）

```
料金構成:
- リクエスト: $0.20 per 1M requests
- 実行時間: $0.0000166667 per GB-second
- コールドスタート: 無料（ただし遅延あり）

月間100万リクエスト、平均100ms、512MB:
- リクエスト料金: $0.20
- 実行時間料金: 1,000,000 × 0.1 × 0.5 × 0.0000166667 = $0.83
- 合計: $1.03
```

#### Provisioned Concurrency あり

```
料金構成:
- Provisioned Concurrency: $0.0000041667 per GB-second
- リクエスト: $0.20 per 1M requests  
- 実行時間: $0.0000166667 per GB-second

同時実行数10、512MB、24時間365日:
- PC料金: 10 × 0.5GB × 2,592,000秒 × 0.0000041667 = $54.00
- リクエスト料金: $0.20
- 実行時間料金: $0.83
- 合計: $55.03

コスト増: 約53倍
```

## 実装例

### 1. オンデマンド Lambda（現在）

```typescript
// 通常のLambda関数
resource "aws_lambda_function" "app" {
  function_name = "attendance-api"
  role          = aws_iam_role.lambda.arn
  handler       = "index.handler"
  runtime       = "nodejs20.x"
  memory_size   = 512
  timeout       = 30

  environment {
    variables = {
      NODE_ENV = "production"
    }
  }
}

// コールドスタート: 初回1-3秒
// ウォーム時: 50-100ms
```

### 2. Provisioned Concurrency

```typescript
// Lambda関数
resource "aws_lambda_function" "app" {
  function_name = "attendance-api"
  role          = aws_iam_role.lambda.arn
  handler       = "index.handler"
  runtime       = "nodejs20.x"
  memory_size   = 512
  timeout       = 30
}

// バージョン公開
resource "aws_lambda_alias" "prod" {
  name             = "prod"
  function_name    = aws_lambda_function.app.function_name
  function_version = aws_lambda_function.app.version
}

// Provisioned Concurrency設定
resource "aws_lambda_provisioned_concurrency_config" "prod" {
  function_name                     = aws_lambda_function.app.function_name
  provisioned_concurrent_executions = 10 // 常時10個の実行環境を維持
  qualifier                         = aws_lambda_alias.prod.name
}

// コールドスタート: なし
// 常時レスポンス: 50-100ms
```

### 3. Application Auto Scaling（動的調整）

```typescript
// 時間帯によって自動調整
resource "aws_appautoscaling_target" "lambda" {
  max_capacity       = 20
  min_capacity       = 5
  resource_id        = "function:${aws_lambda_function.app.function_name}:${aws_lambda_alias.prod.name}"
  scalable_dimension = "lambda:function:ProvisionedConcurrentExecutions"
  service_namespace  = "lambda"
}

resource "aws_appautoscaling_policy" "lambda" {
  name               = "lambda-scaling-policy"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.lambda.resource_id
  scalable_dimension = aws_appautoscaling_target.lambda.scalable_dimension
  service_namespace  = aws_appautoscaling_target.lambda.service_namespace

  target_tracking_scaling_policy_configuration {
    predefined_metric_specification {
      predefined_metric_type = "LambdaProvisionedConcurrencyUtilization"
    }
    target_value = 0.7 // 70%使用率を目標
  }
}

// スケジュールベースのスケーリング
resource "aws_appautoscaling_scheduled_action" "scale_up" {
  name               = "scale-up-morning"
  service_namespace  = aws_appautoscaling_target.lambda.service_namespace
  resource_id        = aws_appautoscaling_target.lambda.resource_id
  scalable_dimension = aws_appautoscaling_target.lambda.scalable_dimension
  schedule           = "cron(0 8 * * ? *)" // 毎朝8時

  scalable_target_action {
    min_capacity = 20
    max_capacity = 50
  }
}
```

## コールドスタート対策の代替案

### 1. Lambda 関数の最適化

```typescript
// グローバルスコープで初期化（再利用される）
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient } from '@aws-sdk/lib-dynamodb';

const client = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client);

export const handler = async (event) => {
  // ハンドラー内は軽量に保つ
  const result = await docClient.send(...);
  return { statusCode: 200, body: JSON.stringify(result) };
};
```

### 2. メモリサイズの最適化

```typescript
// メモリ増加 = CPU増加 = 高速化
resource "aws_lambda_function" "app" {
  memory_size = 1024 // 512MB → 1024MB（2倍高速）
}

// コスト: 2倍
// 実行時間: 半分
// 実質コスト: 同等だが、レイテンシ改善
```

### 3. Lambda SnapStart（Java専用）

Java関数の起動時間を90%削減する機能（Node.jsは未対応）

### 4. 定期的なウォームアップ

```typescript
// CloudWatch Eventsで5分ごとに呼び出し
resource "aws_cloudwatch_event_rule" "warmup" {
  name                = "lambda-warmup"
  schedule_expression = "rate(5 minutes)"
}

resource "aws_cloudwatch_event_target" "lambda" {
  rule      = aws_cloudwatch_event_rule.warmup.name
  target_id = "LambdaWarmup"
  arn       = aws_lambda_function.app.arn
  input     = jsonencode({ warmup: true })
}

// Lambda関数でウォームアップリクエストを無視
export const handler = async (event) => {
  if (event.warmup) {
    return { statusCode: 200, body: 'Warmed up' };
  }
  // 通常処理
};
```

**メリット**: 低コスト  
**デメリット**: 完全にはコールドスタートを防げない

## パフォーマンス比較

| 構成 | 初回レイテンシ | ウォーム時レイテンシ | 月間コスト（100万req） |
|------|---------------|-------------------|---------------------|
| オンデマンド | 1-3秒 | 50-100ms | $1 |
| PC (5並列) | 50-100ms | 50-100ms | $27 |
| PC (10並列) | 50-100ms | 50-100ms | $54 |
| 定期ウォームアップ | 500ms-1秒 | 50-100ms | $2 |

## 推奨構成

### 開発環境
- **Provisioned Concurrency**: なし
- **対策**: コード最適化、メモリ増加

### ステージング環境
- **Provisioned Concurrency**: なし
- **対策**: 定期ウォームアップ（コスト削減）

### 本番環境（低トラフィック）
- **Provisioned Concurrency**: なし
- **対策**: Lambda最適化、メモリ1024MB

### 本番環境（高トラフィック）
- **Provisioned Concurrency**: 5-10並列
- **Auto Scaling**: 有効化

## 導入判断基準

| 指標 | 閾値 | 現状 | 判定 |
|------|------|------|------|
| P99レイテンシ要件 | < 200ms | 1-3秒（初回） | ⚠️ 要検討 |
| 秒間リクエスト | > 100 | < 10 | ✅ 不要 |
| コールドスタート率 | > 10% | < 5% | ✅ 許容範囲 |
| 月間予算 | > $50 | $10 | ❌ 予算超過 |

**結論**: 現状では Provisioned Concurrency は不要。Lambda最適化で対応。

## 学習リソース

- [Lambda Provisioned Concurrency](https://docs.aws.amazon.com/lambda/latest/dg/provisioned-concurrency.html)
- [Managing Concurrency](https://docs.aws.amazon.com/lambda/latest/dg/configuration-concurrency.html)
- [Cold Start Analysis](https://aws.amazon.com/blogs/compute/operating-lambda-performance-optimization-part-1/)

## 関連ドキュメント

- [バックエンドデプロイ方針](../../docs/deploy/BACKEND.md)
- [パフォーマンステスト](../../docs/test/PERFORMANCE.md)

## 更新履歴

- 2024-12-21: 初版作成
