# ファイル階層

このドキュメントは、spec-driven-developmentリポジトリの現在のファイル階層を示します。

## ディレクトリツリー

```
.
├── CONTRIBUTING.md              # コントリビューションガイド（人間向け）
├── LICENSE                      # MITライセンスファイル
├── README.md                    # プロジェクトの概要と目次
├── agent.md                     # AIエージェント向けの作業ガイド
├── docs/                        # プロジェクトドキュメント
│   ├── DEVELOPMENT_FLOW.md      # 開発フローの全体像
│   ├── agreement/               # 合意事項・規約
│   │   ├── code-comment.md      # コードコメント規約
│   │   ├── commit-convention.md # コミットメッセージ規約
│   │   ├── communication.md     # コミュニケーション規約
│   │   ├── diagram-style.md     # 図表作成規約（Mermaid）
│   │   ├── document-structure.md # ドキュメント構造規約
│   │   ├── review-guidelines.md # レビューガイドライン
│   │   ├── work-approach.md     # 作業の進め方
│   │   └── writing-style.md     # 文章スタイル規約
│   ├── build/                   # ビルド戦略
│   │   ├── BACKEND.md           # バックエンドビルド戦略
│   │   ├── CI_CD.md             # CI/CDビルド設定
│   │   ├── FRONTEND.md          # フロントエンドビルド戦略
│   │   ├── README.md            # ビルド戦略の概要
│   │   └── WEBSITE.md           # ドキュメントウェブサイトビルド戦略
│   ├── deploy/                  # デプロイ戦略
│   │   ├── BACKEND.md           # バックエンドデプロイ方針
│   │   ├── CI_CD.md             # CI/CDパイプライン設定
│   │   ├── DATABASE.md          # データベース管理
│   │   ├── FRONTEND.md          # フロントエンドデプロイ方針
│   │   ├── README.md            # デプロイ戦略の概要
│   │   ├── STRATEGIES.md        # デプロイメント戦略（ローリング、ブルーグリーン等）
│   │   └── WEBSITE.md           # ドキュメントウェブサイトデプロイ方針
│   ├── implement/               # 実装仕様
│   │   ├── API.md               # API設計とエンドポイント仕様
│   │   ├── BACKEND.md           # バックエンド設計（NestJS + TypeScript）
│   │   ├── DATABASE.md          # データベース設計（DynamoDB）
│   │   ├── DEPENDENCIES.md      # 依存関係管理（Dependabot、Renovate）
│   │   ├── FRONTEND.md          # フロントエンド設計（React + TypeScript）
│   │   ├── README.md            # 実装仕様の概要
│   │   ├── SECURITY.md          # セキュリティ設計（認証・認可）
│   │   └── WEBSITE.md           # ウェブサイト設計
│   ├── monitor/                 # 監視戦略
│   │   ├── ALERTS.md            # アラート設定
│   │   ├── DASHBOARDS.md        # ダッシュボード設定（Grafana）
│   │   ├── ERRORS.md            # エラー追跡（Sentry）
│   │   ├── LOGGING.md           # ログ管理（Winston/Pino）
│   │   ├── METRICS.md           # メトリクス収集（Prometheus）
│   │   └── README.md            # 監視戦略の概要
│   ├── monorepo/                # モノレポ構成
│   │   ├── DEPENDENCIES.md      # パッケージ間依存関係管理
│   │   ├── README.md            # モノレポ概要
│   │   ├── STRUCTURE.md         # ディレクトリ構造（計画中）
│   │   └── WORKFLOWS.md         # 開発ワークフロー
│   ├── operate/                 # 運用管理
│   │   ├── BACKUP.md            # バックアップとリストア
│   │   ├── DAILY.md             # 日常運用タスク
│   │   ├── INCIDENTS.md         # インシデント対応
│   │   ├── MAINTENANCE.md       # システムメンテナンス
│   │   ├── README.md            # 運用管理の概要
│   │   └── SECURITY.md          # セキュリティ運用
│   ├── release/                 # リリース管理
│   │   ├── CHANGELOG.md         # チェンジログ管理
│   │   ├── NOTES.md             # リリースノート
│   │   ├── PROCESS.md           # リリースプロセス
│   │   ├── README.md            # リリース管理の概要
│   │   └── VERSIONING.md        # バージョニング戦略
│   ├── test/                    # テスト戦略
│   │   ├── ACCESSIBILITY.md     # アクセシビリティテスト（WCAG 2.1 Level AA）
│   │   ├── COMPONENT.md         # コンポーネントテスト（Storybook）
│   │   ├── E2E.md               # E2Eテスト（Cucumber + Playwright）
│   │   ├── INTEGRATION.md       # 統合テスト（Jest + Supertest）
│   │   ├── PERFORMANCE.md       # パフォーマンステスト（Lighthouse CI）
│   │   ├── README.md            # テスト戦略の概要
│   │   ├── SECURITY_DYNAMIC.md  # 動的セキュリティテスト（OWASP ZAP）
│   │   ├── SECURITY_STATIC.md   # 静的セキュリティテスト（npm audit、CodeQL）
│   │   ├── SNAPSHOT.md          # スナップショットテスト（Vitest）
│   │   └── UNIT.md              # 単体テスト（Jest、Vitest）
│   └── user/                    # ユーザー向けドキュメント
│       ├── FAQ.md               # よくある質問と回答
│       ├── MANUAL.md            # ユーザーマニュアル（ナビゲーション）
│       ├── README.md            # 機能リファレンス
│       ├── RELEASE_NOTES.md     # リリースノート
│       ├── guides/              # ガイド
│       │   ├── startup.md       # スタートアップガイド
│       │   └── troubleshooting.md # トラブルシューティング
│       └── images/              # UIスクリーンショット・図表
│           ├── attendance-daily.svg   # 日次勤怠画面
│           ├── attendance-monthly.svg # 月次勤怠画面
│           ├── clock-history.svg      # 打刻履歴画面
│           ├── clock-in.svg           # 出勤打刻画面
│           ├── clock-out.svg          # 退勤打刻画面
│           ├── login.svg              # ログイン画面
│           ├── logout.svg             # ログアウト画面
│           ├── main-dashboard.svg     # メインダッシュボード画面
│           └── user-registration.svg  # ユーザー登録画面
└── specs/                       # 仕様書
    ├── README.md                # 仕様書の概要
    ├── architecture/            # アーキテクチャ決定記録（ADR）
    │   ├── 001-state-management-library.md # 状態管理ライブラリの選定
    │   ├── 002-ui-library-selection.md     # UIライブラリの選定
    │   ├── 003-orm-selection.md            # ORMの選定
    │   ├── 004-redis-cache-usage.md        # Redisキャッシュ利用
    │   ├── 005-cloud-hosting-platform.md   # クラウドホスティングプラットフォーム選定
    │   ├── 006-dynamodb-single-table-design.md # DynamoDBシングルテーブル設計
    │   ├── 007-dynamodb-toolbox-adoption.md    # DynamoDB Toolbox採用
    │   ├── 008-dynamodb-dax-usage.md       # DynamoDB DAX利用
    │   ├── 009-dynamodb-lsi-usage.md       # DynamoDB LSI利用
    │   ├── 010-dynamodb-global-tables.md   # DynamoDBグローバルテーブル
    │   ├── 011-vpc-endpoint-usage.md       # VPCエンドポイント利用
    │   ├── 012-api-gateway-integration.md  # API Gateway統合
    │   ├── 013-lambda-provisioned-concurrency.md # Lambdaプロビジョニング済み同時実行数
    │   ├── 014-s3-local-emulation.md       # S3ローカルエミュレーション
    │   └── README.md                       # アーキテクチャ決定記録の概要
    └── business/                # ビジネス要件仕様
        ├── 001-add-user-registration.md # ユーザー登録機能
        ├── 002-add-login.md             # ログイン機能
        ├── 003-add-logout.md            # ログアウト機能
        ├── 004-add-clock-in-out.md      # 出勤・退勤打刻機能
        ├── 005-add-attendance-history.md # 勤怠履歴機能
        └── images/              # ビジネス要件関連図表
            ├── 001-user-registration.svg # ユーザー登録フロー図
            ├── 002-login.svg             # ログインフロー図
            ├── 003-logout.svg            # ログアウトフロー図
            ├── 004-clock-in.svg          # 出勤打刻フロー図
            ├── 004-clock-out.svg         # 退勤打刻フロー図
            ├── 004-main-dashboard.svg    # メインダッシュボード図
            ├── 005-attendance-daily.svg  # 日次勤怠履歴図
            ├── 005-attendance-monthly.svg # 月次勤怠履歴図
            └── 005-clock-history.svg     # 打刻履歴図
```

## ディレクトリ・ファイル概要

### ルートディレクトリ

| ファイル/ディレクトリ | 説明 |
|---------------------|------|
| `CONTRIBUTING.md` | コントリビューションガイド（人間向け） |
| `LICENSE` | MITライセンスファイル |
| `README.md` | プロジェクトの概要と目次 |
| `agent.md` | AIエージェント向けの作業ガイド |
| `docs/` | プロジェクトドキュメント |
| `specs/` | 仕様書（アーキテクチャとビジネス要件） |

### docs/ - プロジェクトドキュメント

#### docs/agreement/ - 合意事項

プロジェクト全体で合意された規約やガイドライン。

| ファイル | 説明 |
|---------|------|
| `code-comment.md` | コードコメント規約 |
| `commit-convention.md` | コミットメッセージ規約 |
| `communication.md` | コミュニケーション規約 |
| `diagram-style.md` | 図表作成規約（Mermaid） |
| `document-structure.md` | ドキュメント構造規約 |
| `review-guidelines.md` | レビューガイドライン |
| `work-approach.md` | 作業の進め方 |
| `writing-style.md` | 文章スタイル規約 |

#### docs/build/ - ビルド戦略

各コンポーネントのビルド方針と設定。

| ファイル | 説明 |
|---------|------|
| `BACKEND.md` | バックエンドビルド戦略 |
| `CI_CD.md` | CI/CDビルド設定 |
| `FRONTEND.md` | フロントエンドビルド戦略 |
| `README.md` | ビルド戦略の概要 |
| `WEBSITE.md` | ドキュメントウェブサイトビルド戦略 |

#### docs/deploy/ - デプロイ戦略

デプロイメント方針とCI/CD設定。

| ファイル | 説明 |
|---------|------|
| `BACKEND.md` | バックエンドデプロイ方針 |
| `CI_CD.md` | CI/CDパイプライン設定 |
| `DATABASE.md` | データベース管理 |
| `FRONTEND.md` | フロントエンドデプロイ方針 |
| `README.md` | デプロイ戦略の概要 |
| `STRATEGIES.md` | デプロイメント戦略（ローリング、ブルーグリーン等） |
| `WEBSITE.md` | ドキュメントウェブサイトデプロイ方針 |

#### docs/implement/ - 実装仕様

システム実装の詳細設計。

| ファイル | 説明 |
|---------|------|
| `API.md` | API設計とエンドポイント仕様 |
| `BACKEND.md` | バックエンド設計（NestJS + TypeScript） |
| `DATABASE.md` | データベース設計（DynamoDB） |
| `DEPENDENCIES.md` | 依存関係管理（Dependabot、Renovate） |
| `FRONTEND.md` | フロントエンド設計（React + TypeScript） |
| `README.md` | 実装仕様の概要 |
| `SECURITY.md` | セキュリティ設計（認証・認可） |
| `WEBSITE.md` | ウェブサイト設計 |

#### docs/monitor/ - 監視戦略

システム監視とオブザーバビリティ。

| ファイル | 説明 |
|---------|------|
| `ALERTS.md` | アラート設定 |
| `DASHBOARDS.md` | ダッシュボード設定（Grafana） |
| `ERRORS.md` | エラー追跡（Sentry） |
| `LOGGING.md` | ログ管理（Winston/Pino） |
| `METRICS.md` | メトリクス収集（Prometheus） |
| `README.md` | 監視戦略の概要 |

#### docs/monorepo/ - モノレポ構成

モノレポの構造と運用方法。

| ファイル | 説明 |
|---------|------|
| `DEPENDENCIES.md` | パッケージ間依存関係管理 |
| `README.md` | モノレポ概要 |
| `STRUCTURE.md` | ディレクトリ構造（計画中） |
| `WORKFLOWS.md` | 開発ワークフロー |

#### docs/operate/ - 運用管理

日常運用とメンテナンス。

| ファイル | 説明 |
|---------|------|
| `BACKUP.md` | バックアップとリストア |
| `DAILY.md` | 日常運用タスク |
| `INCIDENTS.md` | インシデント対応 |
| `MAINTENANCE.md` | システムメンテナンス |
| `README.md` | 運用管理の概要 |
| `SECURITY.md` | セキュリティ運用 |

#### docs/release/ - リリース管理

バージョン管理とリリースプロセス。

| ファイル | 説明 |
|---------|------|
| `CHANGELOG.md` | チェンジログ管理 |
| `NOTES.md` | リリースノート |
| `PROCESS.md` | リリースプロセス |
| `README.md` | リリース管理の概要 |
| `VERSIONING.md` | バージョニング戦略 |

#### docs/test/ - テスト戦略

各種テストの方針と実装方法。

| ファイル | 説明 |
|---------|------|
| `ACCESSIBILITY.md` | アクセシビリティテスト（WCAG 2.1 Level AA） |
| `COMPONENT.md` | コンポーネントテスト（Storybook） |
| `E2E.md` | E2Eテスト（Cucumber + Playwright） |
| `INTEGRATION.md` | 統合テスト（Jest + Supertest） |
| `PERFORMANCE.md` | パフォーマンステスト（Lighthouse CI） |
| `README.md` | テスト戦略の概要 |
| `SECURITY_DYNAMIC.md` | 動的セキュリティテスト（OWASP ZAP） |
| `SECURITY_STATIC.md` | 静的セキュリティテスト（npm audit、CodeQL） |
| `SNAPSHOT.md` | スナップショットテスト（Vitest） |
| `UNIT.md` | 単体テスト（Jest、Vitest） |

#### docs/user/ - ユーザー向けドキュメント

エンドユーザー向けのマニュアルとガイド。

| ファイル/ディレクトリ | 説明 |
|---------------------|------|
| `FAQ.md` | よくある質問と回答 |
| `MANUAL.md` | ユーザーマニュアル（ナビゲーション） |
| `README.md` | 機能リファレンス |
| `RELEASE_NOTES.md` | リリースノート |
| `guides/startup.md` | スタートアップガイド |
| `guides/troubleshooting.md` | トラブルシューティング |
| `images/` | UIのスクリーンショットやダイアグラム |

#### docs/DEVELOPMENT_FLOW.md

開発フローの全体像を説明するドキュメント。

### specs/ - 仕様書

#### specs/architecture/ - アーキテクチャ決定記録（ADR）

技術選定とアーキテクチャに関する決定事項。

| ファイル | 説明 |
|---------|------|
| `001-state-management-library.md` | 状態管理ライブラリの選定 |
| `002-ui-library-selection.md` | UIライブラリの選定 |
| `003-orm-selection.md` | ORMの選定 |
| `004-redis-cache-usage.md` | Redisキャッシュ利用 |
| `005-cloud-hosting-platform.md` | クラウドホスティングプラットフォーム選定 |
| `006-dynamodb-single-table-design.md` | DynamoDBシングルテーブル設計 |
| `007-dynamodb-toolbox-adoption.md` | DynamoDB Toolbox採用 |
| `008-dynamodb-dax-usage.md` | DynamoDB DAX利用 |
| `009-dynamodb-lsi-usage.md` | DynamoDB LSI利用 |
| `010-dynamodb-global-tables.md` | DynamoDBグローバルテーブル |
| `011-vpc-endpoint-usage.md` | VPCエンドポイント利用 |
| `012-api-gateway-integration.md` | API Gateway統合 |
| `013-lambda-provisioned-concurrency.md` | Lambdaプロビジョニング済み同時実行数 |
| `014-s3-local-emulation.md` | S3ローカルエミュレーション |
| `README.md` | アーキテクチャ決定記録の概要 |

#### specs/business/ - ビジネス要件仕様

勤怠管理システムの機能要件。

| ファイル | 説明 |
|---------|------|
| `001-add-user-registration.md` | ユーザー登録機能 |
| `002-add-login.md` | ログイン機能 |
| `003-add-logout.md` | ログアウト機能 |
| `004-add-clock-in-out.md` | 出勤・退勤打刻機能 |
| `005-add-attendance-history.md` | 勤怠履歴機能 |
| `images/` | ビジネス要件に関連する図表（SVG） |
| `README.md` | ビジネス要件仕様の概要 |

## 統計情報

- **ディレクトリ数**: 18
- **ファイル数**: 109
- **ドキュメント構成**: 
  - 合意事項: 8ファイル
  - ビルド戦略: 5ファイル
  - デプロイ戦略: 7ファイル
  - 実装仕様: 8ファイル
  - 監視戦略: 6ファイル
  - モノレポ構成: 4ファイル
  - 運用管理: 6ファイル
  - リリース管理: 5ファイル
  - テスト戦略: 10ファイル
  - ユーザー向けドキュメント: 6ファイル + 11画像
  - アーキテクチャADR: 15ファイル
  - ビジネス仕様: 6ファイル + 9画像

## プロジェクトの特徴

このリポジトリは、**仕様駆動開発（Spec-Driven Development）** の実践プロジェクトです。AIエージェントとの協働を前提とした、包括的なドキュメント体系を構築しています。

### 主な特徴

1. **ドキュメントファースト**: 実装コードよりも先に、詳細な仕様書とドキュメントを整備
2. **AIエージェント対応**: `agent.md` により、AIエージェントが自律的に作業できる環境を提供
3. **包括的なテスト戦略**: 単体テストからセキュリティテストまで、10種類のテスト戦略を定義
4. **運用を見据えた設計**: 監視、運用、リリース管理まで、プロダクション運用を想定した文書化
5. **アーキテクチャの透明性**: ADR（Architecture Decision Records）による技術選定の記録

---

**最終更新日**: 2025年12月  
**リポジトリ**: [goataka/spec-driven-development](https://github.com/goataka/spec-driven-development)
