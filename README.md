# Document-Driven Development

## 目的

このリポジトリは、**AIエージェントによる開発**と**ドキュメント駆動開発（Document-Driven Development）** の実践・検証を行うプロジェクトです。

ドキュメント駆動開発とは、実装に先立ってドキュメントを作成し、システムの仕様や動作を明確にすることで、より質の高い開発を目指す手法です。このリポジトリでは、AIエージェントがこの手法を用いて対象システムの開発を行い、その有効性を検証します。

## コントリビューション

このプロジェクトは**AIエージェントのみが実装を行う**実験的なリポジトリです。

- **実装・コード変更**: AIエージェントのみが行います
- **人間の貢献**: 指示の提供とレビューを通じて行います

人間の貢献方法の詳細は[CONTRIBUTING.md](CONTRIBUTING.md)をご覧ください。

## 対象システム

勤怠管理システム

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

## AIエージェント向け指示

AIエージェントがこのリポジトリで作業する際のガイドラインを[agent.md](agent.md)に記載しています。

## ライセンス

このプロジェクトはMITライセンスの下で公開されています。詳細は[LICENSE](LICENSE)ファイルをご覧ください。

## お問い合わせ

質問や提案がある場合は、GitHubのIssueセクションにて投稿してください。
