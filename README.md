# 勤怠管理システム - Document-Driven Development

## 概要

このリポジトリは、**ドキュメント駆動開発（Document-Driven Development）** の実践例として、勤怠管理システムの開発を行うプロジェクトです。

ドキュメント駆動開発とは、実装に先立ってドキュメントを作成し、システムの仕様や動作を明確にすることで、より質の高い開発を目指す手法です。

## プロジェクト構成

```
.
├── README.md                     # このファイル
├── spec/                         # 技術仕様ディレクトリ
│   ├── ARCHITECTURE.md           # システムアーキテクチャ設計書（概要）
│   ├── FRONTEND.md               # フロントエンド設計詳細
│   ├── BACKEND.md                # バックエンド設計詳細
│   ├── DATABASE.md               # データベース設計詳細
│   ├── API.md                    # API設計詳細
│   ├── SECURITY.md               # セキュリティ設計詳細
│   └── DEPENDENCIES.md           # 依存関係管理と自動更新
├── test/                         # テスト戦略ディレクトリ
│   ├── TESTING.md                # テスト戦略（概要・単体・コンポーネント）
│   ├── TESTING_INTEGRATION.md    # 統合・E2Eテスト
│   ├── TESTING_SNAPSHOT.md       # スナップショットテスト
│   ├── TESTING_ACCESSIBILITY.md  # アクセシビリティテスト（WCAG 2.1）
│   ├── TESTING_PERFORMANCE.md    # パフォーマンステスト（Lighthouse CI）
│   ├── TESTING_SECURITY.md       # 静的セキュリティテスト
│   └── TESTING_SECURITY_DYNAMIC.md # 動的セキュリティテスト（OWASP ZAP）
├── docs/                         # ユーザードキュメントディレクトリ
│   ├── MANUAL.md                # マニュアルナビゲーション
│   ├── RELEASE_NOTES.md         # リリースノート
│   ├── FAQ.md                   # よくある質問
│   ├── guides/                  # ガイド
│   │   ├── startup.md          # スタートアップガイド
│   │   └── troubleshooting.md  # トラブルシューティング
│   ├── reference/               # 機能リファレンス
│   │   ├── user-registration.md # ユーザー登録機能
│   │   ├── login.md            # ログイン機能
│   │   ├── logout.md           # ログアウト機能
│   │   ├── clock-in-out.md     # 出勤・退勤打刻機能
│   │   └── attendance-history.md # 勤怠履歴機能
│   └── images/                  # ドキュメント用画像
│       ├── user-registration.svg
│       ├── login-screen.svg
│       ├── main-dashboard.svg
│       ├── clock-in.svg
│       ├── clock-out.svg
│       ├── attendance-history.svg
│       └── logout.svg
├── CONTRIBUTING.md               # コントリビューションガイド
└── LICENSE                       # ライセンス情報
```

## ドキュメント

### 技術仕様（spec/）

#### 🏗️ [システムアーキテクチャ設計書](spec/ARCHITECTURE.md)
システム全体のアーキテクチャ概要と設計方針。React + NestJSベースの構成について解説。

#### 📱 [フロントエンド設計](spec/FRONTEND.md)
React + TypeScriptの詳細設計とコンポーネント実装例。

#### 🔧 [バックエンド設計](spec/BACKEND.md)
NestJS + TypeScriptの詳細設計とサービス実装例。

#### 💾 [データベース設計](spec/DATABASE.md)
PostgreSQLのテーブル設計とマイグレーション戦略。

#### 🌐 [API設計](spec/API.md)
RESTful APIのエンドポイント仕様とレスポンス形式。

#### 🔒 [セキュリティ](spec/SECURITY.md)
認証・認可とセキュリティ対策の詳細。

#### 🔗 [依存関係管理](spec/DEPENDENCIES.md)
Dependabot、Renovateによる自動更新（すべて無料）。

### テスト戦略（test/）

#### ✅ [テスト戦略（概要）](test/TESTING.md)
テスト戦略の概要と基本テスト（単体テスト・コンポーネントテスト・Storybook）。

##### 詳細なテストドキュメント:
- [統合・E2Eテスト](test/TESTING_INTEGRATION.md) - Jest + Supertest統合テスト、Cucumber + Playwright E2Eテスト
- [スナップショットテスト](test/TESTING_SNAPSHOT.md) - Vitestスナップショット、視覚的回帰テスト
- [アクセシビリティテスト](test/TESTING_ACCESSIBILITY.md) - WCAG 2.1 Level AA準拠、jest-axe
- [パフォーマンステスト](test/TESTING_PERFORMANCE.md) - Lighthouse CI、Core Web Vitals
- [静的セキュリティテスト](test/TESTING_SECURITY.md) - npm audit、audit-ci、GitHub CodeQL（すべて無料）
- [動的セキュリティテスト](test/TESTING_SECURITY_DYNAMIC.md) - OWASP ZAP、インジェクション攻撃テスト（すべて無料）

### ユーザードキュメント（docs/）

#### 📘 [ユーザーマニュアル](docs/MANUAL.md)
各種ドキュメントへのナビゲーションページ。目的に応じた適切なドキュメントを案内します。

### 📖 [スタートアップガイド](docs/guides/startup.md)
初めてシステムを使う方向けの基本的な使い方ガイド。利用の流れに沿って説明しています。

### 📚 機能リファレンス
各機能の詳細な使い方を個別に解説：
- [ユーザー登録機能](docs/reference/user-registration.md)
- [ログイン機能](docs/reference/login.md)
- [ログアウト機能](docs/reference/logout.md)
- [出勤・退勤打刻機能](docs/reference/clock-in-out.md)
- [勤怠履歴機能](docs/reference/attendance-history.md)

### 🔧 [トラブルシューティング](docs/guides/troubleshooting.md)
よくある問題と解決方法をカテゴリ別に整理。

### ❓ [FAQ](docs/FAQ.md)
システムに関するよくある質問と回答。

### 📋 [リリースノート](docs/RELEASE_NOTES.md)
システムの機能概要、バージョン情報、今後の予定など。

## システム要件

本システムは以下の機能を提供します：

### 主要機能

1. **認証機能**
   - ユーザー登録（メールアドレス、パスワード、会社コード）
   - メールアドレスとパスワードによるログイン
   - セキュアなセッション管理

2. **勤怠打刻機能**
   - 出勤時刻の記録
   - 退勤時刻の記録

3. **勤怠履歴機能**
   - ログインユーザー自身の打刻履歴の閲覧
   - 期間指定での検索
   - 勤務時間の自動計算

## 開発の流れ

このプロジェクトはドキュメント駆動開発に基づいており、以下の順序で進めます：

1. ✅ **ドキュメント作成**（完了）
   - リリースノートの作成
   - ユーザーマニュアルの作成
   - 画面イメージの作成

2. ✅ **アーキテクチャ設計**（完了）
   - システムアーキテクチャ設計
   - 技術スタックの選定
   - 開発方針の策定

3. 🔄 **要件定義と詳細設計**（次のフェーズ）
   - 機能要件の詳細化
   - 非機能要件の定義
   - API仕様の策定
   - データベース詳細設計
   - UI/UX詳細設計

4. ⏳ **実装**（予定）
   - バックエンド開発
   - フロントエンド開発
   - テスト作成

5. ⏳ **テスト・デプロイ**（予定）
   - 統合テスト
   - ユーザー受け入れテスト
   - 本番環境へのデプロイ

## コントリビューション

このプロジェクトへの貢献を歓迎します。詳細は[CONTRIBUTING.md](CONTRIBUTING.md)をご覧ください。

## ライセンス

このプロジェクトはMITライセンスの下で公開されています。詳細は[LICENSE](LICENSE)ファイルをご覧ください。

## お問い合わせ

質問や提案がある場合は、GitHubのIssueセクションにて投稿してください。