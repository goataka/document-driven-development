# Spec-Driven Development

**AIエージェント**との**仕様駆動開発（Spec-Driven Development）** を実践・検証するリポジトリです。

## コントリビューション

| 対象 | 役割 | 詳細 |
|------|------|------|
| **人間** | 指示・レビュー | [CONTRIBUTING.md](CONTRIBUTING.md) |
| **AIエージェント** | 実装・テスト・運用 | [agent.md](agent.md) |

## 対象システム

勤怠管理システム

## アーキテクチャ決定記録（ADR）

アーキテクチャに関する重要な決定事項と検討中の項目については、[specs/architecture/](specs/architecture/)ディレクトリを参照してください。

## ドキュメント

### 開発の流れ

詳細は[開発の流れ](docs/DEVELOPMENT_FLOW.md)をご覧ください。

### 合意事項（docs/agreement/）

#### 📝 [作業の進め方](docs/agreement/work-approach.md)

変更の進め方と基本原則。

#### 💬 [コードコメント規約](docs/agreement/code-comment.md)

統一されたコードコメントの記述方法。

#### 📋 [コミット規約](docs/agreement/commit-convention.md)

統一されたコミットメッセージのフォーマット。

#### 🗣️ [コミュニケーション規約](docs/agreement/communication.md)

プロジェクト内でのコミュニケーション方法。

#### 📊 [図表作成規約](docs/agreement/diagram-style.md)

Mermaidを使用した図表の作成方法。

#### 📄 [ドキュメント構造規約](docs/agreement/document-structure.md)

統一されたドキュメントの構造と記述方法。

#### 🔍 [レビューガイドライン](docs/agreement/review-guidelines.md)

コードレビューとドキュメントレビューの観点。

#### ✍️ [文章スタイル規約](docs/agreement/writing-style.md)

統一された文章スタイル。

### [実装仕様](docs/implement/)

#### 📱 [フロントエンド設計](docs/implement/FRONTEND.md)

React + TypeScriptの詳細設計とコンポーネント実装例。

#### 🔧 [バックエンド設計](docs/implement/BACKEND.md)

NestJS + TypeScriptの詳細設計とサービス実装例。

#### 💾 [データベース設計](docs/implement/DATABASE.md)
DynamoDBのテーブル設計とデータモデリング戦略。

#### 🌐 [API設計](docs/implement/API.md)

RESTful APIのエンドポイント仕様とレスポンス形式。

#### 🔒 [セキュリティ](docs/implement/SECURITY.md)

認証・認可とセキュリティ対策の詳細。

#### 🔗 [依存関係管理](docs/implement/DEPENDENCIES.md)

Dependabot、Renovateによる自動更新（すべて無料）。

#### 📄 [ウェブサイト設計](docs/implement/WEBSITE.md)

ドキュメントwebsiteとアプリケーションの連携、ヘルプボタン実装。

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

#### 📄 [websiteビルド](docs/build/WEBSITE.md)
ドキュメントwebsiteのビルド方針、MkDocs/VitePress設定。

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
データモデル設計、バックアップ戦略。

#### 🎯 [デプロイメント戦略](docs/deploy/STRATEGIES.md)
ローリングデプロイ、ブルーグリーンデプロイ、カナリアリリース。

#### 🔄 [CI/CD パイプライン](docs/deploy/CI_CD.md)
GitHub Actionsデプロイ設定、環境変数管理。

#### 📄 [websiteデプロイ](docs/deploy/WEBSITE.md)
ドキュメントwebsiteのホスティング、GitHub Pages/Vercel/Netlifyデプロイ。

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

#### 📘 [ユーザーマニュアル](docs/user/MANUAL.md)

各種ドキュメントへのナビゲーションページ。目的に応じた適切なドキュメントを案内します。

#### 📖 [スタートアップガイド](docs/user/guides/startup.md)

初めてシステムを使う方向けの基本的な使い方ガイド。利用の流れに沿って説明しています。

#### 📚 [機能リファレンス](docs/user/README.md)

各機能の詳細な使い方（ユーザー登録、ログイン、ログアウト、出勤・退勤打刻、勤怠履歴）。

#### 🔧 [トラブルシューティング](docs/user/guides/troubleshooting.md)

よくある問題と解決方法をカテゴリ別に整理。

#### ❓ [FAQ](docs/user/FAQ.md)

システムに関するよくある質問と回答。

#### 📋 [リリースノート](docs/user/RELEASE_NOTES.md)

システムの機能概要、バージョン情報、今後の予定など。

## [モノレポ構成](docs/monorepo/)

#### 📂 [モノレポ概要](docs/monorepo/README.md)

モノレポ採用理由、メリット・デメリット、適用シナリオ。

#### 🗂️ [ディレクトリ構造](docs/monorepo/STRUCTURE.md)

apps/、packages/、static-sites/の詳細構成、命名規則。

#### 🔄 [ワークフロー](docs/monorepo/WORKFLOWS.md)

開発フロー、CI/CD設定、Turborepoによるタスク実行。

#### 🔗 [依存関係管理](docs/monorepo/DEPENDENCIES.md)

パッケージ間の依存関係、バージョン管理、セキュリティ対策。

## ライセンス

このプロジェクトはMITライセンスの下で公開されています。詳細は[LICENSE](LICENSE)ファイルをご覧ください。

## お問い合わせ

質問や提案がある場合は、GitHubのIssueセクションにて投稿してください。
