# 勤怠管理システム - Document-Driven Development

## 概要

このリポジトリは、**ドキュメント駆動開発（Document-Driven Development）** の実践例として、勤怠管理システムの開発を行うプロジェクトです。

ドキュメント駆動開発とは、実装に先立ってドキュメントを作成し、システムの仕様や動作を明確にすることで、より質の高い開発を目指す手法です。

## ドキュメント

### 開発の流れ

詳細は[開発の流れ](docs/DEVELOPMENT_FLOW.md)をご覧ください。

### [実装仕様](docs/implement/)

#### 📱 [フロントエンド設計](docs/implement/FRONTEND.md)
React + TypeScriptの詳細設計とコンポーネント実装例。

#### 🔧 [バックエンド設計](docs/implement/BACKEND.md)
NestJS + TypeScriptの詳細設計とサービス実装例。

#### 💾 [データベース設計](docs/implement/DATABASE.md)
PostgreSQLのテーブル設計とマイグレーション戦略。

#### 🌐 [API設計](docs/implement/API.md)
RESTful APIのエンドポイント仕様とレスポンス形式。

#### 🔒 [セキュリティ](docs/implement/SECURITY.md)
認証・認可とセキュリティ対策の詳細。

#### 🔗 [依存関係管理](docs/implement/DEPENDENCIES.md)
Dependabot、Renovateによる自動更新（すべて無料）。

## [テスト戦略](docs/test/)

#### 🧪 [単体テスト](docs/test/UNIT.md)
Jest（backend）、Vitest + React Testing Library（frontend）。

#### 🎨 [コンポーネントテスト](docs/test/COMPONENT.md)
Storybook、ビジュアルテスト。

#### 🔗 [統合テスト](docs/test/INTEGRATION.md)
Jest + Supertest APIテスト。

#### 🎭 [E2Eテスト](docs/test/E2E.md)
Cucumber + Playwright、BDDテスト。

#### 📸 [スナップショットテスト](docs/test/SNAPSHOT.md)
Vitestスナップショット、視覚的回帰テスト。

#### ♿ [アクセシビリティテスト](docs/test/ACCESSIBILITY.md)
WCAG 2.1 Level AA準拠、jest-axe。

#### ⚡ [パフォーマンステスト](docs/test/PERFORMANCE.md)
Lighthouse CI、Core Web Vitals。

#### 🔐 [静的セキュリティテスト](docs/test/SECURITY_STATIC.md)
npm audit、audit-ci、GitHub CodeQL（すべて無料）。

#### 🛡️ [動的セキュリティテスト](docs/test/SECURITY_DYNAMIC.md)
OWASP ZAP、インジェクション攻撃テスト（すべて無料）。

## [ビルド戦略](docs/build/)

#### 📱 [フロントエンドビルド](docs/build/FRONTEND.md)
フロントエンドのビルド方針、最適化戦略、環境別設定方針。

#### 🔧 [バックエンドビルド](docs/build/BACKEND.md)
バックエンドのビルド方針、Lambda対応バンドル戦略、最適化方針。

#### 🔄 [CI/CDビルド](docs/build/CI_CD.md)
GitHub Actionsでのビルド方針、キャッシュ戦略。

## [リリース管理](docs/release/)

#### 🏷️ [バージョニング戦略](docs/release/VERSIONING.md)
セマンティックバージョニング、バージョン決定基準。

#### 🔄 [リリースプロセス](docs/release/PROCESS.md)
リリース計画、リリース手順、自動化されたリリース。

#### 📝 [チェンジログ管理](docs/release/CHANGELOG.md)
CHANGELOG.md の構造、コミットメッセージ規約。

#### 📄 [リリースノート](docs/release/NOTES.md)
リリースノートのテンプレート、作成方法。

## [デプロイ戦略](docs/deploy/)

#### 📱 [フロントエンドデプロイ](docs/deploy/FRONTEND.md)
Vercel、Netlify、AWS S3 + CloudFrontへのデプロイ。

#### 🔧 [バックエンドデプロイ](docs/deploy/BACKEND.md)
Docker コンテナ化、Heroku、AWS ECS/Fargateへのデプロイ。

#### 💾 [データベース管理](docs/deploy/DATABASE.md)
マイグレーション実行、バックアップ戦略。

#### 🎯 [デプロイメント戦略](docs/deploy/STRATEGIES.md)
ローリングデプロイ、ブルーグリーンデプロイ、カナリアリリース。

#### 🔄 [CI/CD パイプライン](docs/deploy/CI_CD.md)
GitHub Actionsデプロイ設定、環境変数管理。

## [運用管理](docs/operate/)

#### 📅 [日常運用タスク](docs/operate/DAILY.md)
毎日のチェックリスト、システムヘルスチェック。

#### 🔧 [システムメンテナンス](docs/operate/MAINTENANCE.md)
定期メンテナンス、データベースメンテナンス。

#### 💾 [バックアップとリストア](docs/operate/BACKUP.md)
バックアップ戦略、自動バックアップ設定、リストア手順。

#### 🔒 [セキュリティ運用](docs/operate/SECURITY.md)
セキュリティチェックリスト、セキュリティインシデント対応。

#### 🚨 [インシデント対応](docs/operate/INCIDENTS.md)
インシデント管理フロー、インシデントレポート。

## [監視戦略](docs/monitor/)

#### 📝 [ログ管理](docs/monitor/LOGGING.md)
ログレベル、Winston/Pino設定、ログローテーション。

#### 📊 [メトリクス収集](docs/monitor/METRICS.md)
Prometheus設定、カスタムメトリクス。

#### 🔔 [アラート設定](docs/monitor/ALERTS.md)
アラートルール、Alertmanager設定、通知設定。

#### 🐛 [エラー追跡](docs/monitor/ERRORS.md)
Sentry設定、エラーレポート、スタックトレース分析。

#### 📈 [ダッシュボード](docs/monitor/DASHBOARDS.md)
Grafanaダッシュボード設定、可視化パネル。

## [ユーザー向けドキュメント](docs/user/)

ユーザーマニュアル、スタートアップガイド、機能リファレンス、FAQ、リリースノートなど、システムの利用に関する全てのドキュメント。

## AIエージェント向け指示

AIエージェントがこのリポジトリで作業する際のガイドラインを[agent.md](agent.md)に記載しています。

## コントリビューション

このプロジェクトへの貢献を歓迎します。詳細は[CONTRIBUTING.md](CONTRIBUTING.md)をご覧ください。

## ライセンス

このプロジェクトはMITライセンスの下で公開されています。詳細は[LICENSE](LICENSE)ファイルをご覧ください。

## お問い合わせ

質問や提案がある場合は、GitHubのIssueセクションにて投稿してください。
