# websiteアプリケーション連携

本ドキュメントは、アプリケーション機能内からwebsiteへのリンク方法を説明します。

## 概要

アプリ内の「ヘルプ」ボタンや機能説明から、関連するマニュアルページへユーザーを誘導します。

### 連携の目的

- アプリ内からコンテキストに応じた適切なマニュアルページへ誘導
- リリースノートで新機能の詳細情報を提供
- トラブルシューティングガイドへの簡単なアクセス

---

## 実装方法

### 1. ヘルプボタンコンポーネント

各画面にヘルプボタンを配置し、該当機能のマニュアルへリンク：

```typescript
// React コンポーネント例
import React from 'react';

interface HelpButtonProps {
  docPath: string;  // マニュアルページのパス
  label?: string;
}

export const HelpButton: React.FC<HelpButtonProps> = ({ 
  docPath, 
  label = 'ヘルプ' 
}) => {
  const docsBaseUrl = process.env.REACT_APP_DOCS_URL || 'https://docs.example.com';
  const fullUrl = `${docsBaseUrl}/${docPath}`;
  
  return (
    <a
      href={fullUrl}
      target="_blank"
      rel="noopener noreferrer"
      className="help-button"
      aria-label={`${label}を開く（新しいタブ）`}
    >
      <HelpIcon />
      {label}
    </a>
  );
};
```

### 2. 機能別のリンク設定

```typescript
// helpLinks.ts
export const helpLinks = {
  login: 'reference/login',
  userRegistration: 'reference/user-registration',
  clockIn: 'reference/clock-in-out#出勤打刻',
  clockOut: 'reference/clock-in-out#退勤打刻',
  attendanceHistory: 'reference/attendance-history',
  troubleshooting: 'guides/troubleshooting',
  faq: 'FAQ',
} as const;

// 使用例
<HelpButton docPath={helpLinks.login} />
```

### 3. コンテキストヘルプ

画面ごとに適切なヘルプコンテンツを表示：

```typescript
// ContextHelp.tsx
import React from 'react';
import { useLocation } from 'react-router-dom';
import { HelpButton } from './HelpButton';
import { helpLinks } from './helpLinks';

const routeToHelpMap: Record<string, string> = {
  '/login': helpLinks.login,
  '/register': helpLinks.userRegistration,
  '/dashboard': helpLinks.clockIn,
  '/attendance': helpLinks.attendanceHistory,
};

export const ContextHelp: React.FC = () => {
  const location = useLocation();
  const helpPath = routeToHelpMap[location.pathname];
  
  if (!helpPath) return null;
  
  return <HelpButton docPath={helpPath} />;
};
```

---

## リリースノート連携

### 1. 新機能通知

新バージョンリリース時に通知を表示：

```typescript
// ReleaseNotification.tsx
import React, { useState, useEffect } from 'react';

interface Release {
  version: string;
  date: string;
  url: string;
}

export const ReleaseNotification: React.FC = () => {
  const [latestRelease, setLatestRelease] = useState<Release | null>(null);
  const [dismissed, setDismissed] = useState(false);
  
  useEffect(() => {
    // リリース情報を取得
    fetchLatestRelease().then(release => {
      const lastSeenVersion = localStorage.getItem('lastSeenVersion');
      if (release.version !== lastSeenVersion) {
        setLatestRelease(release);
      }
    });
  }, []);
  
  const handleDismiss = () => {
    if (latestRelease) {
      localStorage.setItem('lastSeenVersion', latestRelease.version);
      setDismissed(true);
    }
  };
  
  if (!latestRelease || dismissed) return null;
  
  return (
    <div className="release-notification">
      <p>
        新しいバージョン {latestRelease.version} がリリースされました！
      </p>
      <a 
        href={latestRelease.url}
        target="_blank"
        rel="noopener noreferrer"
      >
        詳細を見る
      </a>
      <button onClick={handleDismiss}>閉じる</button>
    </div>
  );
};
```

### 2. バージョン情報表示

フッターにバージョンとリリースノートへのリンクを表示：

```typescript
// Footer.tsx
import React from 'react';

export const Footer: React.FC = () => {
  const version = process.env.REACT_APP_VERSION || '1.0.0';
  const releaseNotesUrl = `${process.env.REACT_APP_DOCS_URL}/versions/v${version}`;
  
  return (
    <footer className="app-footer">
      <div className="footer-content">
        <span>バージョン {version}</span>
        <a 
          href={releaseNotesUrl}
          target="_blank"
          rel="noopener noreferrer"
        >
          リリースノート
        </a>
      </div>
    </footer>
  );
};
```

---

## エラーメッセージからのリンク

エラー発生時にトラブルシューティングガイドへリンク：

```typescript
// ErrorBoundary.tsx
import React, { Component, ErrorInfo, ReactNode } from 'react';

interface Props {
  children: ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      const troubleshootingUrl = `${process.env.REACT_APP_DOCS_URL}/guides/troubleshooting`;
      
      return (
        <div className="error-page">
          <h1>エラーが発生しました</h1>
          <p>申し訳ございません。予期しないエラーが発生しました。</p>
          <div className="error-actions">
            <button onClick={() => window.location.reload()}>
              ページを再読み込み
            </button>
            <a 
              href={troubleshootingUrl}
              target="_blank"
              rel="noopener noreferrer"
            >
              トラブルシューティングガイド
            </a>
          </div>
        </div>
      );
    }

    return this.props.children;
  }
}
```

---

## ディープリンク

マニュアルページの特定セクションへ直接リンク：

```typescript
// ディープリンクの生成
const createDeepLink = (docPath: string, anchor?: string): string => {
  const baseUrl = process.env.REACT_APP_DOCS_URL || 'https://docs.example.com';
  return anchor ? `${baseUrl}/${docPath}#${anchor}` : `${baseUrl}/${docPath}`;
};

// 使用例
export const ClockInHelp: React.FC = () => {
  return (
    <div>
      <h2>出勤打刻</h2>
      <p>
        出勤打刻の詳しい手順は
        <a 
          href={createDeepLink('reference/clock-in-out', '出勤打刻')}
          target="_blank"
          rel="noopener noreferrer"
        >
          こちら
        </a>
        をご覧ください。
      </p>
    </div>
  );
};
```

---

## 環境変数設定

各環境でドキュメントURLを設定：

```bash
# .env.development
REACT_APP_DOCS_URL=http://localhost:8000

# .env.staging
REACT_APP_DOCS_URL=https://staging-docs.example.com

# .env.production
REACT_APP_DOCS_URL=https://docs.example.com
```

---

## バックエンドAPI経由でのリンク提供

APIレスポンスにマニュアルリンクを含める：

```typescript
// Backend (NestJS) 例
@Controller('api/help')
export class HelpController {
  private readonly docsBaseUrl = process.env.DOCS_URL || 'https://docs.example.com';
  
  @Get('links')
  getHelpLinks() {
    return {
      login: `${this.docsBaseUrl}/reference/login`,
      registration: `${this.docsBaseUrl}/reference/user-registration`,
      clockIn: `${this.docsBaseUrl}/reference/clock-in-out#出勤打刻`,
      attendance: `${this.docsBaseUrl}/reference/attendance-history`,
    };
  }
  
  @Get('latest-release')
  getLatestRelease() {
    return {
      version: '1.0.0',
      releaseDate: '2025-12-21',
      releaseNotesUrl: `${this.docsBaseUrl}/versions/v1.0.0`,
    };
  }
}
```

---

## アクセシビリティ

### スクリーンリーダー対応

```typescript
<a
  href={docsUrl}
  target="_blank"
  rel="noopener noreferrer"
  aria-label="ログイン機能のヘルプ（新しいタブで開きます）"
>
  ヘルプ
</a>
```

### キーボードナビゲーション

```typescript
<button
  onClick={openDocs}
  onKeyPress={(e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      openDocs();
    }
  }}
  aria-label="マニュアルを開く"
>
  ヘルプ
</button>
```

---

## まとめ

アプリケーションとwebsiteを効果的に連携させることで：

1. ユーザーが必要な情報に素早くアクセスできる
2. コンテキストに応じた適切なヘルプを提供できる
3. 新機能やリリース情報を効果的に伝えられる
4. サポート負荷を軽減できる

---

**最終更新日**: 2025年12月  
**バージョン**: 1.0.0
