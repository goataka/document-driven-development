# 勤怠管理システム - Document-Driven Development

## 概要

このリポジトリは、**ドキュメント駆動開発（Document-Driven Development）** の実践例として、勤怠管理システムの開発を行うプロジェクトです。

ドキュメント駆動開発とは、実装に先立ってドキュメントを作成し、システムの仕様や動作を明確にすることで、より質の高い開発を目指す手法です。

## ドキュメント

### 開発の流れ

詳細は[開発の流れ](docs/DEVELOPMENT_FLOW.md)をご覧ください。

### 実装仕様（docs/implement/）

#### 🏗️ [システムアーキテクチャ設計書](docs/implement/ARCHITECTURE.md)
システム全体のアーキテクチャ概要と設計方針。React + NestJSベースの構成について解説。

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

## テスト戦略（docs/test/）

#### ✅ [テスト戦略概要](docs/test/OVERVIEW.md)
テスト戦略の全体像とテストピラミッド。

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

#### 🔐 [静的セキュリティテスト](docs/test/SECURITY.md)
npm audit、audit-ci、GitHub CodeQL（すべて無料）。

#### 🛡️ [動的セキュリティテスト](docs/test/SECURITY_DYNAMIC.md)
OWASP ZAP、インジェクション攻撃テスト（すべて無料）。

## ユーザー向けドキュメント（docs/user/）

#### 📘 [ユーザーマニュアル](docs/user/MANUAL.md)
各種ドキュメントへのナビゲーションページ。目的に応じた適切なドキュメントを案内。

#### 📖 [スタートアップガイド](docs/user/guides/startup.md)
初めてシステムを使う方向けの基本的な使い方ガイド。

#### 📚 [機能リファレンス（ユーザー登録）](docs/user/reference/user-registration.md)
ユーザー登録機能の詳細な使い方。

#### 📚 [機能リファレンス（ログイン）](docs/user/reference/login.md)
ログイン機能の詳細な使い方。

#### 📚 [機能リファレンス（ログアウト）](docs/user/reference/logout.md)
ログアウト機能の詳細な使い方。

#### 📚 [機能リファレンス（出勤・退勤打刻）](docs/user/reference/clock-in-out.md)
出勤・退勤打刻機能の詳細な使い方。

#### 📚 [機能リファレンス（勤怠履歴）](docs/user/reference/attendance-history.md)
勤怠履歴機能の詳細な使い方。

#### 🔧 [トラブルシューティング](docs/user/guides/troubleshooting.md)
よくある問題と解決方法をカテゴリ別に整理。

#### ❓ [FAQ](docs/user/FAQ.md)
システムに関するよくある質問と回答。

#### 📋 [リリースノート](docs/user/RELEASE_NOTES.md)
システムの機能概要、バージョン情報、今後の予定など。

## コントリビューション

このプロジェクトへの貢献を歓迎します。詳細は[CONTRIBUTING.md](CONTRIBUTING.md)をご覧ください。

## ライセンス

このプロジェクトはMITライセンスの下で公開されています。詳細は[LICENSE](LICENSE)ファイルをご覧ください。

## お問い合わせ

質問や提案がある場合は、GitHubのIssueセクションにて投稿してください。
