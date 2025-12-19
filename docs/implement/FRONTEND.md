# フロントエンド設計

本ドキュメントは、React + TypeScriptをベースとしたフロントエンド設計の詳細を説明します。

**関連ドキュメント**: [システムアーキテクチャ設計書](../implement/ARCHITECTURE.md)

## 目次

1. [ディレクトリ構成](#ディレクトリ構成)
2. [主要設計パターン](#主要設計パターン)
3. [実装例](#実装例)

---

## ディレクトリ構成

```
frontend/
├── public/                    # 静的ファイル
│   └── favicon.ico
├── src/
│   ├── main.tsx              # エントリーポイント
│   ├── App.tsx               # ルートコンポーネント
│   ├── routes/               # ルーティング設定
│   │   └── index.tsx
│   ├── pages/                # ページコンポーネント
│   │   ├── Login/
│   │   ├── Register/
│   │   ├── Dashboard/
│   │   ├── ClockInOut/
│   │   └── AttendanceHistory/
│   ├── components/           # 共通コンポーネント
│   │   ├── common/           # 汎用コンポーネント
│   │   │   ├── Button/
│   │   │   ├── Input/
│   │   │   └── Modal/
│   │   └── layouts/          # レイアウトコンポーネント
│   │       ├── Header/
│   │       ├── Sidebar/
│   │       └── Footer/
│   ├── hooks/                # カスタムフック
│   │   ├── useAuth.ts
│   │   └── useAttendance.ts
│   ├── store/                # 状態管理
│   │   ├── authStore.ts
│   │   └── attendanceStore.ts
│   ├── services/             # API通信
│   │   ├── api.ts            # Axios設定
│   │   ├── authService.ts
│   │   └── attendanceService.ts
│   ├── types/                # TypeScript型定義
│   │   ├── user.ts
│   │   └── attendance.ts
│   ├── utils/                # ユーティリティ関数
│   │   ├── dateFormatter.ts
│   │   └── validator.ts
│   ├── constants/            # 定数
│   │   └── index.ts
│   └── styles/               # グローバルスタイル
│       └── global.css
├── package.json
├── tsconfig.json
├── vite.config.ts
└── .env.example
```

---

## 主要設計パターン

### 1. コンポーネント設計

#### 原則

- **Atomic Design**の考え方を部分的に採用
- **関数コンポーネント + React Hooks**を使用
- **プレゼンテーションコンポーネント**と**コンテナコンポーネント**を分離

#### 実装例: Buttonコンポーネント

```typescript
// src/components/common/Button/Button.tsx
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
}

export const Button: React.FC<ButtonProps> = ({
  label,
  onClick,
  variant = 'primary',
  disabled = false
}) => {
  return (
    <button
      className={`btn btn-${variant}`}
      onClick={onClick}
      disabled={disabled}
    >
      {label}
    </button>
  );
};
```

---

### 2. 状態管理

#### 原則

- **ローカル状態**: useState, useReducer
- **グローバル状態**: Zustand または Redux Toolkit
- **サーバー状態**: React Query（TanStack Query）の導入を推奨

#### 実装例: Zustandストア

```typescript
// src/store/authStore.ts
import { create } from 'zustand';

interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

export const useAuthStore = create<AuthState>((set) => ({
  user: null,
  token: null,
  isAuthenticated: false,
  login: async (email, password) => {
    const { user, token } = await authService.login(email, password);
    set({ user, token, isAuthenticated: true });
  },
  logout: () => {
    set({ user: null, token: null, isAuthenticated: false });
  }
}));
```

---

### 3. ルーティング

#### 原則

- React Routerを使用したクライアントサイドルーティング
- ProtectedRouteコンポーネントで認証制御
- ネストされたルート構造

#### 実装例: ルート定義

```typescript
// src/routes/index.tsx
import { createBrowserRouter, Navigate } from 'react-router-dom';
import { Layout } from '../components/layouts/Layout';
import { Login } from '../pages/Login';
import { Register } from '../pages/Register';
import { Dashboard } from '../pages/Dashboard';
import { ClockInOut } from '../pages/ClockInOut';
import { AttendanceHistory } from '../pages/AttendanceHistory';
import { ProtectedRoute } from '../components/ProtectedRoute';

export const router = createBrowserRouter([
  {
    path: '/',
    element: <Layout />,
    children: [
      { index: true, element: <Navigate to="/dashboard" replace /> },
      { path: 'login', element: <Login /> },
      { path: 'register', element: <Register /> },
      {
        path: 'dashboard',
        element: <ProtectedRoute><Dashboard /></ProtectedRoute>
      },
      {
        path: 'clock',
        element: <ProtectedRoute><ClockInOut /></ProtectedRoute>
      },
      {
        path: 'history',
        element: <ProtectedRoute><AttendanceHistory /></ProtectedRoute>
      }
    ]
  }
]);
```

---

### 4. API通信

#### 原則

- Axiosインスタンスの作成と設定
- インターセプターによる共通処理（トークン付与、エラーハンドリング）
- 型安全なAPI呼び出し

#### 実装例: Axios設定

```typescript
// src/services/api.ts
import axios from 'axios';

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL || 'http://localhost:3000/api',
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json'
  }
});

// リクエストインターセプター（トークン付与）
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// レスポンスインターセプター（エラーハンドリング）
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // トークン無効 -> ログアウト処理
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default api;
```

---

## 実装例

詳細な実装例については、各セクションに記載されています。

**関連ドキュメント**:
- [システムアーキテクチャ設計書](../implement/ARCHITECTURE.md)
- [バックエンド設計](./BACKEND.md)
- [テスト戦略](./TESTING.md)

---

**最終更新日**: 2024年12月17日  
**バージョン**: 1.0.0
