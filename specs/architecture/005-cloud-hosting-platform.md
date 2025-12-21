# ADR-005: クラウドホスティングプラットフォームの選定

## ステータス

提案中（Proposed）

## 背景

本番環境のホスティングプラットフォームを選定する必要があります。要件としては：

- コスト効率
- 運用負荷の低減
- スケーラビリティ
- セキュリティ
- デプロイの容易さ

以下のプラットフォームを検討しています：

### フロントエンド
- Vercel
- Netlify
- AWS S3 + CloudFront
- AWS Amplify

### バックエンド
- AWS Lambda + API Gateway
- Heroku
- AWS ECS/Fargate
- GCP Cloud Run
- Azure App Service

### データベース
- AWS DynamoDB
- Heroku Postgres
- AWS RDS
- Supabase

## 決定事項

### 推奨構成

**フロントエンド**: Vercel（第一選択）
- 理由: SPAに最適、自動デプロイ、CDN標準装備、無料プランあり

**バックエンド**: AWS Lambda + API Gateway
- 理由: サーバーレス、自動スケーリング、従量課金

**データベース**: AWS DynamoDB
- 理由: フルマネージド、高可用性、コスト効率

### 代替案

#### フロントエンド代替案

**Netlify**
- 用途: Vercelと同等の機能が必要な場合
- メリット: 豊富なプラグイン、フォーム機能
- デメリット: ビルド時間制限

**AWS S3 + CloudFront**
- 用途: AWS環境に統一したい場合
- メリット: AWSサービスとの統合
- デメリット: 設定の複雑さ

#### バックエンド代替案

**Heroku**
- 用途: 簡単なデプロイを優先する場合
- メリット: 設定が簡単、開発者体験が良い
- デメリット: コストが高い

**AWS ECS/Fargate**
- 用途: コンテナベースのデプロイが必要な場合
- メリット: Docker対応、柔軟性
- デメリット: 設定の複雑さ

**GCP Cloud Run**
- 用途: GCP環境を利用する場合
- メリット: サーバーレスコンテナ、自動スケーリング
- デメリット: AWS DynamoDBとの連携が複雑

## 結果

### メリット（推奨構成の場合）

**コスト効率**:
- Vercel: 無料プラン〜月額$20
- Lambda: 従量課金（月額$5-20想定）
- DynamoDB: 従量課金（月額$15-60想定）
- **合計**: 月額$20-100（トラフィック次第）

**運用負荷**:
- フルマネージドサービス
- 自動スケーリング
- 高可用性（SLA 99.9%以上）

**開発体験**:
- Git連携による自動デプロイ
- プレビュー環境の自動作成
- ロールバックが容易

### デメリット

**ベンダーロックイン**:
- AWSへの依存度が高い
- 移行コストが発生する可能性

**学習コスト**:
- Lambda + API Gatewayの設定に習熟が必要
- DynamoDBのデータモデリング学習

**コールドスタート**:
- Lambda関数の初回起動遅延
- Provisioned Concurrencyで対策可能（追加コスト）

### 段階的移行戦略

1. **Phase 1（初期）**: Vercel + Heroku + Heroku Postgres
   - 迅速な立ち上げ
   - 低い学習コスト
   - コスト: 月額$20-40

2. **Phase 2（中期）**: Vercel + AWS Lambda + DynamoDB
   - スケーラビリティ向上
   - コスト最適化
   - コスト: 月額$20-60

3. **Phase 3（長期）**: AWS構成への統一（必要に応じて）
   - S3 + CloudFront + Lambda + DynamoDB
   - 完全なAWSエコシステム活用

## 環境別構成

### 開発環境
- ローカルマシン（Docker Compose）
- コスト: $0

### ステージング環境
- Vercel（Preview Deployments）
- Lambda（最小構成）
- DynamoDB（オンデマンドモード）
- コスト: 月額$5-15

### 本番環境
- Vercel（Production）
- Lambda + API Gateway
- DynamoDB（オンデマンド or プロビジョニング）
- コスト: 月額$20-100

## セキュリティ考慮事項

- HTTPS必須（全環境）
- IAM ロールベースのアクセス制御
- VPC設定（必要に応じて）
- AWS Secrets Manager でシークレット管理
- DDoS対策（CloudFront, API Gateway）

## パフォーマンス考慮事項

- CDN活用（Vercel, CloudFront）
- Lambda関数のメモリ・タイムアウト最適化
- DynamoDBのキャパシティ設定
- CloudWatch によるモニタリング

## 関連ドキュメント

- [システムアーキテクチャ設計書](../../docs/implement/README.md)
- [フロントエンドデプロイ](../../docs/deploy/FRONTEND.md)
- [バックエンドデプロイ](../../docs/deploy/BACKEND.md)
- [データベース管理](../../docs/deploy/DATABASE.md)

## 更新履歴

- 2024-12-21: 初版作成
