# 勤怠管理システム - Document-Driven Development

## 概要

このリポジトリは、**ドキュメント駆動開発（Document-Driven Development）** の実践例として、勤怠管理システムの開発を行うプロジェクトです。

ドキュメント駆動開発とは、実装に先立ってドキュメントを作成し、システムの仕様や動作を明確にすることで、より質の高い開発を目指す手法です。

## プロジェクト構成

```
.
├── README.md                     # このファイル
├── docs/                         # ドキュメントディレクトリ
│   ├── implement/                # 実装仕様ディレクトリ
│   │   ├── README.md            # システムアーキテクチャ設計書（概要）
│   │   ├── FRONTEND.md          # フロントエンド設計詳細
│   │   ├── BACKEND.md           # バックエンド設計詳細
│   │   ├── DATABASE.md          # データベース設計詳細
│   │   ├── API.md               # API設計詳細
│   │   ├── SECURITY.md          # セキュリティ設計詳細
│   │   └── DEPENDENCIES.md      # 依存関係管理と自動更新
│   ├── test/                    # テスト戦略ディレクトリ
│   │   ├── README.md            # テスト戦略概要
│   │   ├── UNIT.md              # 単体テスト
│   │   ├── COMPONENT.md         # コンポーネントテスト（Storybook）
│   │   ├── INTEGRATION.md       # 統合テスト
│   │   ├── E2E.md               # E2Eテスト（Cucumber + Playwright）
│   │   ├── SNAPSHOT.md          # スナップショットテスト
│   │   ├── ACCESSIBILITY.md     # アクセシビリティテスト（WCAG 2.1）
│   │   ├── PERFORMANCE.md       # パフォーマンステスト（Lighthouse CI）
│   │   ├── SECURITY.md          # 静的セキュリティテスト
│   │   └── SECURITY_DYNAMIC.md  # 動的セキュリティテスト（OWASP ZAP）
│   └── user/                    # ユーザーマニュアル
│       ├── README.md           # ユーザー向けドキュメント一覧
│       ├── MANUAL.md           # マニュアルナビゲーション
│       ├── RELEASE_NOTES.md    # リリースノート
│       ├── FAQ.md              # よくある質問
│       ├── guides/              # ガイド
│       │   ├── startup.md      # スタートアップガイド
│       │   └── troubleshooting.md # トラブルシューティング
│       ├── reference/           # 機能リファレンス
│       │   ├── user-registration.md # ユーザー登録機能
│       │   ├── login.md        # ログイン機能
│       │   ├── logout.md       # ログアウト機能
│       │   ├── clock-in-out.md # 出勤・退勤打刻機能
│       │   └── attendance-history.md # 勤怠履歴機能
│       └── images/              # ドキュメント用画像
│           ├── user-registration.svg
│           ├── login.svg
│           ├── main-dashboard.svg
│           ├── clock-in.svg
│           ├── clock-out.svg
│           ├── attendance-history.svg
│           └── logout.svg
├── CONTRIBUTING.md               # コントリビューションガイド
└── LICENSE                       # ライセンス情報
```

## ドキュメント

### 実装仕様（docs/implement/）

#### 🏗️ [システムアーキテクチャ設計書](docs/implement/)
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

### ✅ テスト戦略（docs/test/）

#### ✅ [テスト戦略概要](docs/test/)
テスト戦略の全体像とテストピラミッド。

- [単体テスト](docs/test/UNIT.md) - Jest（backend）、Vitest + React Testing Library（frontend）
- [コンポーネントテスト](docs/test/COMPONENT.md) - Storybook、ビジュアルテスト
- [統合テスト](docs/test/INTEGRATION.md) - Jest + Supertest APIテスト
- [E2Eテスト](docs/test/E2E.md) - Cucumber + Playwright、BDDテスト
- [スナップショットテスト](docs/test/SNAPSHOT.md) - Vitestスナップショット、視覚的回帰テスト
- [アクセシビリティテスト](docs/test/ACCESSIBILITY.md) - WCAG 2.1 Level AA準拠、jest-axe
- [パフォーマンステスト](docs/test/PERFORMANCE.md) - Lighthouse CI、Core Web Vitals
- [静的セキュリティテスト](docs/test/SECURITY.md) - npm audit、audit-ci、GitHub CodeQL（すべて無料）
- [動的セキュリティテスト](docs/test/SECURITY_DYNAMIC.md) - OWASP ZAP、インジェクション攻撃テスト（すべて無料）

### 📘 [ユーザー向けドキュメント](docs/user/)

ユーザーマニュアル、スタートアップガイド、機能リファレンス、FAQ、リリースノートなど、システムの利用に関する全てのドキュメント。

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
