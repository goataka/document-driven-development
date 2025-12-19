# 単体テスト

本ドキュメントは、単体テストの実装方法を説明します。

**関連ドキュメント**: 
- [テスト戦略概要](./OVERVIEW.md)
- [コンポーネントテスト](./COMPONENT.md)
- [システムアーキテクチャ設計書](../implement/ARCHITECTURE.md)

---


## 単体テスト

### フロントエンド: Vitest + React Testing Library

**対象**:
- ユーティリティ関数
- カスタムフック
- 状態管理ロジック
- 個別のReactコンポーネント

#### Vitest設定

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './src/test/setup.ts',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'src/test/',
      ]
    }
  }
});
```

#### フックテスト例

```typescript
// src/hooks/__tests__/useAuth.test.ts
import { renderHook, act } from '@testing-library/react';
import { describe, it, expect, beforeEach } from 'vitest';
import { useAuthStore } from '../useAuth';

describe('useAuth', () => {
  beforeEach(() => {
    // ストアをリセット
    useAuthStore.getState().logout();
  });

  it('初期状態では認証されていない', () => {
    const { result } = renderHook(() => useAuthStore());
    expect(result.current.isAuthenticated).toBe(false);
    expect(result.current.user).toBeNull();
  });

  it('ログイン後は認証状態になる', async () => {
    const { result } = renderHook(() => useAuthStore());
    
    await act(async () => {
      await result.current.login('test@example.com', 'password123');
    });

    expect(result.current.isAuthenticated).toBe(true);
    expect(result.current.user).toBeDefined();
  });
});
```

##

---

## コンポーネントテスト例

```typescript
// src/components/Button/__tests__/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { Button } from '../Button';

describe('Button', () => {
  it('ラベルが正しく表示される', () => {
    render(<Button label="クリック" onClick={() => {}} />);
    expect(screen.getByText('クリック')).toBeInTheDocument();
  });

  it('クリックイベントが発火する', () => {
    const handleClick = vi.fn();
    render(<Button label="クリック" onClick={handleClick} />);
    
    fireEvent.click(screen.getByText('クリック'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('disabled状態では動作しない', () => {
    const handleClick = vi.fn();
    render(<Button label="クリック" onClick={handleClick} disabled />);
    
    const button = screen.getByText('クリック');
    expect(button).toBeDisabled();
    fireEvent.click(button);
    expect(handleClick).not.toHaveBeenCalled();
  });
});
```

---

### バックエンド: Jest

**対象**:
- サービス層のビジネスロジック
- ユーティリティ関数
- バリデーションロジック

#### サービステスト例

```typescript
// src/modules/attendance/attendance.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { AttendanceService } from './attendance.service';
import { getRepositoryToken } from '@nestjs/typeorm';
import { Attendance } from './entities/attendance.entity';
import { BadRequestException } from '@nestjs/common';
import { Repository } from 'typeorm';

describe('AttendanceService', () => {
  let service: AttendanceService;
  let mockRepository: jest.Mocked<Partial<Repository<Attendance>>>;

  beforeEach(async () => {
    mockRepository = {
      findOne: jest.fn(),
      create: jest.fn(),
      save: jest.fn(),
      find: jest.fn(),
    };

    const module: TestingModule = await Test.createTestingModule({
      providers: [
        AttendanceService,
        {
          provide: getRepositoryToken(Attendance),
          useValue: mockRepository,
        },
      ],
    }).compile();

    service = module.get<AttendanceService>(AttendanceService);
  });

  describe('clockIn', () => {
    it('出勤打刻が正常に記録される', async () => {
      const userId = 'user-123';
      const now = new Date();
      mockRepository.findOne.mockResolvedValue(null);
      mockRepository.create.mockReturnValue({ userId, clockInTime: now } as Partial<Attendance>);
      mockRepository.save.mockResolvedValue({ id: '1', userId, clockInTime: now } as Attendance);

      const result = await service.clockIn(userId);

      expect(result).toBeDefined();
      expect(result.userId).toBe(userId);
      expect(mockRepository.save).toHaveBeenCalled();
    });

    it('既に出勤済みの場合はエラーを投げる', async () => {
      const userId = 'user-123';
      mockRepository.findOne.mockResolvedValue({ userId, clockInTime: new Date() } as Attendance);

      await expect(service.clockIn(userId)).rejects.toThrow(BadRequestException);
    });
  });
});
```

---

