# ADR-011: VPCエンドポイントの利用

## ステータス

提案中（Proposed）

## 背景

DynamoDBやS3などのAWSサービスへのアクセスを、パブリックインターネットを経由せず、AWS内部ネットワーク経由で行うため、VPCエンドポイントの導入を検討しています。

### VPCエンドポイントとは

VPC内のリソースがAWSサービスにプライベートに接続できるようにする機能で、インターネットゲートウェイ、NATデバイス、VPN接続、AWS Direct Connect接続を必要としません。

### VPCエンドポイントの種類

1. **Gateway エンドポイント**: S3、DynamoDB用（無料）
2. **Interface エンドポイント**: その他のAWSサービス用（有料）

## 決定事項

**オプション**: セキュリティとネットワーク最適化が必要な場合にVPCエンドポイントを導入する。

### VPCエンドポイントを導入すべきケース

1. **セキュリティ要件**: トラフィックをAWS内部に限定したい
2. **コンプライアンス**: データがインターネットを経由しない要件
3. **ネットワーク最適化**: レイテンシ削減、帯域幅向上
4. **Lambda in VPC**: VPC内のLambdaからDynamoDB/S3へのアクセス
5. **コスト削減**: NATゲートウェイのデータ転送料削減

### VPCエンドポイントなしで十分なケース

1. **Lambda公開実行**: VPC外でLambdaを実行する場合
2. **シンプルな構成**: セキュリティ要件が低い
3. **開発環境**: ローカル開発、CI環境

## 結果

### メリット（VPCエンドポイント導入）

- **セキュリティ向上**: トラフィックがAWS内部に限定
- **レイテンシ削減**: AWS内部ネットワーク経由
- **コスト削減**: NATゲートウェイのデータ転送料削減（DynamoDB、S3の場合）
- **スループット向上**: インターネット帯域制限を回避
- **コンプライアンス**: データ主権、プライバシー要件への対応

### デメリット

- **設定の複雑さ**: VPC設定が必要
- **追加コスト**: Interface エンドポイント利用時（$0.01/時間 + データ転送料）
- **DNS設定**: プライベートDNSの有効化が必要

### コスト分析

#### Gateway エンドポイント（DynamoDB、S3）
- **コスト**: $0（無料）
- **データ転送料**: 削減効果あり

#### Interface エンドポイント（その他サービス）
- **エンドポイント料金**: $0.01/時間 × 730時間 = $7.30/月
- **データ処理料金**: $0.01/GB
- **合計**: 月額 $7.30 + データ転送量

#### NATゲートウェイなしの場合
- **NATゲートウェイ料金**: $0.045/時間 × 730時間 = $32.85/月
- **データ転送料金**: $0.045/GB
- **合計**: 月額 $32.85 + データ転送量

**結論**: VPCエンドポイント（Gateway）でNATゲートウェイコストを削減可能

## 実装例

### 1. Gateway エンドポイント（DynamoDB）の作成

```hcl
// Terraform例
resource "aws_vpc_endpoint" "dynamodb" {
  vpc_id            = aws_vpc.main.id
  service_name      = "com.amazonaws.ap-northeast-1.dynamodb"
  vpc_endpoint_type = "Gateway"
  
  route_table_ids = [
    aws_route_table.private.id,
  ]

  tags = {
    Name = "dynamodb-vpc-endpoint"
  }
}
```

### 2. Interface エンドポイント（Secrets Manager）の作成

```hcl
// Terraform例
resource "aws_vpc_endpoint" "secretsmanager" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.ap-northeast-1.secretsmanager"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = aws_subnet.private[*].id
  security_group_ids  = [aws_security_group.vpc_endpoint.id]
  
  private_dns_enabled = true

  tags = {
    Name = "secretsmanager-vpc-endpoint"
  }
}
```

### 3. Lambda 関数からの利用

```hcl
// Lambda関数をVPC内に配置
resource "aws_lambda_function" "app" {
  function_name = "attendance-api"
  role          = aws_iam_role.lambda.arn
  
  vpc_config {
    subnet_ids         = aws_subnet.private[*].id
    security_group_ids = [aws_security_group.lambda.id]
  }

  environment {
    variables = {
      DYNAMODB_ENDPOINT = "https://dynamodb.ap-northeast-1.amazonaws.com" // VPCエンドポイント経由
    }
  }
}
```

### 4. アプリケーションコード（変更なし）

```typescript
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, GetCommand } from '@aws-sdk/lib-dynamodb';

// VPCエンドポイント経由で自動的にアクセス（コード変更不要）
const client = new DynamoDBClient({ region: 'ap-northeast-1' });
const docClient = DynamoDBDocumentClient.from(client);

const user = await docClient.send(
  new GetCommand({
    TableName: 'users',
    Key: { userId: '123' },
  })
);
```

## アーキテクチャ比較

### VPCエンドポイントなし

```
Lambda (VPC外)
    ↓ (インターネット経由)
DynamoDB (AWS Managed)
```

### VPCエンドポイントあり

```
Lambda (VPC内)
    ↓ (AWS内部ネットワーク経由)
VPC Endpoint
    ↓
DynamoDB (AWS Managed)
```

## セキュリティ考慮事項

### VPCエンドポイントポリシー

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:Query"
      ],
      "Resource": "arn:aws:dynamodb:ap-northeast-1:123456789012:table/users"
    }
  ]
}
```

### セキュリティグループ（Interface エンドポイント）

```hcl
resource "aws_security_group" "vpc_endpoint" {
  name   = "vpc-endpoint-sg"
  vpc_id = aws_vpc.main.id

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = [aws_vpc.main.cidr_block]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

## パフォーマンス比較

| 構成 | レイテンシ | 帯域幅 | コスト |
|------|-----------|--------|--------|
| インターネット経由 | 10-20ms | 制限あり | NATゲートウェイ料金 |
| VPC Endpoint (Gateway) | 5-10ms | 高速 | $0 |
| VPC Endpoint (Interface) | 5-10ms | 高速 | $7.30/月 |

## 推奨構成

### 開発環境
- **VPCエンドポイント**: 不要
- **Lambda**: VPC外で実行
- **コスト**: 最小

### ステージング環境
- **VPCエンドポイント**: Gateway（DynamoDB、S3）
- **Lambda**: VPC内で実行（本番同等）
- **コスト**: 中程度

### 本番環境
- **VPCエンドポイント**: Gateway + Interface（必要に応じて）
- **Lambda**: VPC内で実行
- **セキュリティ**: 最大化
- **コスト**: 高いがセキュアな構成

## 段階的導入戦略

### Phase 1（現在）: VPCエンドポイントなし
- Lambda をVPC外で実行
- シンプルな構成

### Phase 2（中期）: Gateway エンドポイント導入
- DynamoDB、S3用のGateway エンドポイント
- NATゲートウェイコスト削減

### Phase 3（長期）: Interface エンドポイント追加
- Secrets Manager、CloudWatch Logs等
- 完全なVPC内通信

## 監視とトラブルシューティング

### VPCエンドポイント接続確認

```bash
# VPCエンドポイントDNS解決確認
nslookup dynamodb.ap-northeast-1.amazonaws.com

# VPCエンドポイント経由のアクセステスト
aws dynamodb list-tables --region ap-northeast-1 --endpoint-url https://vpce-xxx.dynamodb.ap-northeast-1.vpce.amazonaws.com
```

### CloudWatch メトリクス

- **PacketCount**: VPCエンドポイント経由のパケット数
- **BytesProcessed**: 処理されたデータ量

## 学習リソース

- [AWS VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html)
- [Gateway Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/vpce-gateway.html)
- [Interface Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/vpce-interface.html)
- [VPC Endpoint Pricing](https://aws.amazon.com/privatelink/pricing/)

## 関連ドキュメント

- [データベース管理方針](../../docs/deploy/DATABASE.md)
- [バックエンドデプロイ方針](../../docs/deploy/BACKEND.md)
- [セキュリティ](../../docs/implement/SECURITY.md)

## 更新履歴

- 2024-12-21: 初版作成
