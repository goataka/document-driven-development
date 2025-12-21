# 静的サイト - サイト構造

このドキュメントは、静的サイトのディレクトリ構成とナビゲーション設計を説明します。

## サイト構成の概要

静的サイトは以下の2つに分けて構築します：

1. **ユーザーマニュアルサイト**: エンドユーザー向け
2. **リリースノートサイト**: バージョン情報と更新履歴

## ユーザーマニュアルサイト構造

### ディレクトリ構成

```
static-sites/user-docs/
├── mkdocs.yml              # MkDocs設定ファイル
├── docs/
│   ├── index.md            # トップページ
│   ├── guides/
│   │   ├── startup.md      # スタートアップガイド
│   │   └── troubleshooting.md  # トラブルシューティング
│   ├── reference/
│   │   ├── user-registration.md
│   │   ├── login.md
│   │   ├── logout.md
│   │   ├── clock-in-out.md
│   │   └── attendance-history.md
│   ├── FAQ.md              # よくある質問
│   └── images/             # 画像ファイル
│       ├── login-screen.png
│       ├── dashboard.png
│       └── ...
└── site/                   # ビルド後の静的ファイル（自動生成）
```

### ナビゲーション構成

```yaml
# mkdocs.yml
nav:
  - ホーム: index.md
  - はじめに:
    - スタートアップガイド: guides/startup.md
  - 機能リファレンス:
    - ユーザー登録: reference/user-registration.md
    - ログイン: reference/login.md
    - ログアウト: reference/logout.md
    - 出勤・退勤打刻: reference/clock-in-out.md
    - 勤怠履歴: reference/attendance-history.md
  - サポート:
    - FAQ: FAQ.md
    - トラブルシューティング: guides/troubleshooting.md
```

### トップページ（index.md）の構成

```markdown
# 勤怠管理システム ユーザーマニュアル

## はじめての方へ

初めてシステムを使う方は、[スタートアップガイド](guides/startup.md)をご覧ください。

## クイックリンク

- [ログイン方法](reference/login.md)
- [出勤・退勤の打刻](reference/clock-in-out.md)
- [勤怠履歴の確認](reference/attendance-history.md)

## サポート

問題が発生した場合:
- [トラブルシューティング](guides/troubleshooting.md)
- [FAQ](FAQ.md)
```

## リリースノートサイト構造

### ディレクトリ構成

```
static-sites/release-notes/
├── mkdocs.yml              # MkDocs設定ファイル
├── docs/
│   ├── index.md            # 最新リリース情報
│   ├── versions/
│   │   ├── v1.0.0.md       # バージョン 1.0.0
│   │   ├── v1.1.0.md       # バージョン 1.1.0
│   │   └── v1.2.0.md       # バージョン 1.2.0
│   ├── changelog.md        # 全変更履歴
│   └── roadmap.md          # 今後の予定
└── site/                   # ビルド後の静的ファイル（自動生成）
```

### ナビゲーション構成

```yaml
# mkdocs.yml
nav:
  - 最新情報: index.md
  - バージョン:
    - v1.2.0: versions/v1.2.0.md
    - v1.1.0: versions/v1.1.0.md
    - v1.0.0: versions/v1.0.0.md
  - 全変更履歴: changelog.md
  - ロードマップ: roadmap.md
```

## ページレイアウトの統一

### 共通ヘッダー

すべてのページに共通のヘッダーを配置：

```markdown
# ページタイトル

> 最終更新日: 2025年12月21日
```

### 共通フッター

すべてのページに共通のフッターを配置：

```markdown
---

## 関連ドキュメント

- [ドキュメントA](../path/to/doc-a.md)
- [ドキュメントB](../path/to/doc-b.md)

## サポート

問題が発生した場合は、[トラブルシューティング](../guides/troubleshooting.md)をご確認ください。
```

## 検索機能

MkDocsの検索機能を有効化：

```yaml
# mkdocs.yml
theme:
  features:
    - search.suggest    # 検索候補の表示
    - search.highlight  # 検索結果のハイライト
    - search.share      # 検索結果の共有

plugins:
  - search:
      lang: ja          # 日本語検索
      separator: '[\s\-\.]+'
```

## ブレッドクラム

ページの階層構造を示すブレッドクラム：

```yaml
# mkdocs.yml
theme:
  features:
    - navigation.breadcrumbs  # ブレッドクラムを表示
```

## サイドバーナビゲーション

### 折りたたみ可能なセクション

```yaml
# mkdocs.yml
theme:
  features:
    - navigation.sections   # セクションを明示
    - navigation.expand     # デフォルトで展開
    - navigation.indexes    # インデックスページ
```

### 目次（Table of Contents）

各ページの右側に目次を表示：

```yaml
# mkdocs.yml
theme:
  features:
    - toc.integrate   # サイドバーに統合
    - toc.follow      # スクロールに追従

markdown_extensions:
  - toc:
      permalink: true    # 見出しへのパーマリンク
      toc_depth: 3       # 深さレベル3まで
```

## モバイル対応

### レスポンシブデザイン

Material for MkDocsはデフォルトでレスポンシブ：

```yaml
# mkdocs.yml
theme:
  name: material
  features:
    - navigation.tabs.sticky  # タブを固定
    - navigation.top          # トップへ戻るボタン
```

### モバイルナビゲーション

小画面ではハンバーガーメニューを自動表示

## テーマカスタマイズ

### カラースキーム

```yaml
# mkdocs.yml
theme:
  palette:
    # ライトモード
    - media: "(prefers-color-scheme: light)"
      scheme: default
      primary: indigo
      accent: indigo
      toggle:
        icon: material/brightness-7
        name: ダークモードに切り替え
    
    # ダークモード
    - media: "(prefers-color-scheme: dark)"
      scheme: slate
      primary: indigo
      accent: indigo
      toggle:
        icon: material/brightness-4
        name: ライトモードに切り替え
```

### カスタムCSS

```yaml
# mkdocs.yml
extra_css:
  - stylesheets/extra.css
```

```css
/* docs/stylesheets/extra.css */
.md-header {
  background-color: #1976d2;
}

.md-typeset h1 {
  color: #1976d2;
}
```

## 多言語対応（将来対応）

```yaml
# mkdocs.yml
plugins:
  - i18n:
      default_language: ja
      languages:
        ja:
          name: 日本語
        en:
          name: English
```

ディレクトリ構成：
```
docs/
├── ja/
│   ├── index.md
│   └── guides/
└── en/
    ├── index.md
    └── guides/
```

## バージョニング

### バージョン別ドキュメント

```yaml
# mkdocs.yml
extra:
  version:
    provider: mike
    default: latest

plugins:
  - mike:
      version_selector: true
```

デプロイコマンド：
```bash
# バージョン 1.0 をデプロイ
mike deploy 1.0 latest --update-aliases

# バージョン一覧
mike list
```

## アクセシビリティ

### WCAG 2.1 Level AA準拠

```yaml
# mkdocs.yml
theme:
  features:
    - content.code.annotate  # コードの注釈
    - content.tabs.link      # タブのリンク

markdown_extensions:
  - attr_list    # HTML属性の追加
  - md_in_html   # HTML内のMarkdown
```

### キーボードナビゲーション

- タブキーでの移動
- エンターキーでのリンク選択
- ESCキーでの検索クローズ

## サイトマップ

自動生成されるサイトマップ：

```yaml
# mkdocs.yml
plugins:
  - sitemap:
      changefreq: weekly
      priority: 0.8
```

生成されるファイル: `site/sitemap.xml`

## RSS/Atomフィード（リリースノート用）

```yaml
# mkdocs.yml
plugins:
  - rss:
      match_path: versions/.*
      date_from_meta:
        as_creation: date
      categories:
        - tags
```

## パフォーマンス最適化

### 画像の最適化

- WebP形式を使用
- 適切なサイズにリサイズ
- 遅延読み込み（lazy loading）

### 静的ファイルの圧縮

```yaml
# GitHub Actionsでの圧縮例
- name: Compress site
  run: |
    find site -name "*.html" -exec gzip -k {} \;
    find site -name "*.css" -exec gzip -k {} \;
    find site -name "*.js" -exec gzip -k {} \;
```

---

**最終更新日**: 2025年12月  
**バージョン**: 1.0.0
