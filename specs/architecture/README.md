# アーキテクチャ決定記録 (Architecture Decision Records)

このディレクトリには、システムアーキテクチャに関する重要な決定事項と検討中の項目を記録します。

## ADRとは

Architecture Decision Records（ADR）は、アーキテクチャ上の重要な決定を記録するための文書形式です。各ADRには以下の情報が含まれます：

- **タイトル**: 決定事項の簡潔な説明
- **ステータス**: 提案中（Proposed）、承認済み（Accepted）、却下（Rejected）、非推奨（Deprecated）、置き換え済み（Superseded）
- **背景**: 決定が必要になった理由や状況
- **決定事項**: 採用した（または検討中の）アプローチ
- **結果**: 決定による影響、メリット、デメリット

## ADR一覧

### フロントエンド

- [001: 状態管理ライブラリの選定](./001-state-management-library.md) - Zustand vs Redux Toolkit
- [002: UIライブラリの選定](./002-ui-library-selection.md) - Material-UI vs その他

### バックエンド

- [003: ORMの選定](./003-orm-selection.md) - TypeORM vs Prisma
- [004: Redisキャッシュの導入](./004-redis-cache-usage.md) - セッション管理とキャッシュ戦略
- [013: Lambda Provisioned Concurrencyの導入](./013-lambda-provisioned-concurrency.md) - コールドスタート対策

### インフラストラクチャ

- [005: クラウドホスティングプラットフォームの選定](./005-cloud-hosting-platform.md) - AWS/GCP/Azure/Vercel/Heroku
- [012: API Gatewayの統合](./012-api-gateway-integration.md) - APIエンドポイント管理

### データベース（DynamoDB）

- [006: DynamoDB Single Table Designの採用](./006-dynamodb-single-table-design.md) - テーブル設計戦略
- [007: DynamoDB Toolboxの導入](./007-dynamodb-toolbox-adoption.md) - データモデリング支援
- [008: DynamoDB DAX（Accelerator）の導入](./008-dynamodb-dax-usage.md) - 高頻度読み取り最適化
- [009: DynamoDB Local Secondary Index（LSI）の利用](./009-dynamodb-lsi-usage.md) - クエリパターン最適化
- [010: DynamoDB グローバルテーブルの構成](./010-dynamodb-global-tables.md) - 多地域展開戦略
- [011: VPCエンドポイントの利用](./011-vpc-endpoint-usage.md) - セキュリティとネットワーク最適化

### 開発環境

- [014: S3ローカルエミュレーション戦略](./014-s3-local-emulation.md) - ローカル開発環境

## ステータスの定義

- **提案中（Proposed）**: 検討中で、まだ決定されていない
- **承認済み（Accepted）**: 決定が承認され、実装中または実装済み
- **却下（Rejected）**: 検討の結果、採用しないことが決定
- **非推奨（Deprecated）**: 過去に承認されたが、現在は推奨されない
- **置き換え済み（Superseded）**: 別のADRによって置き換えられた

## 参考資料

- [ADR GitHub Organization](https://adr.github.io/)
- [Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)

---

**最終更新日**: 2024-12-21  
**バージョン**: 1.0.0
