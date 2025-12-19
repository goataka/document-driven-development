# コードコメント規約

このドキュメントは、プロジェクト全体で統一されたコードコメントの記述方法を定義します。

## コメントの原則

- 過剰なコメントは避ける
- コードで意図が明確な場合はコメント不要
- 複雑なロジックにのみコメントを追加する

## コメントが必要な場合

- アルゴリズムの説明が必要な複雑な処理
- 外部APIやライブラリの特殊な使い方
- ビジネスロジックの背景や理由

## コメントが不要な場合

- 変数名や関数名で意図が明確な場合
- 標準的なパターンやイディオム
- 自明な処理や簡単な計算

## 例：適切なコメント

```typescript
// ❌ 過剰なコメント
// ユーザーIDを取得する
const userId = user.id;

// ユーザー名を取得する
const userName = user.name;

// ❌ 不要なコメント
function add(a: number, b: number): number {
  // aとbを足し算する
  return a + b;
}

// ✅ 適切なコメント
// JWT トークンの有効期限は24時間
// セキュリティポリシーに基づき変更禁止
const TOKEN_EXPIRY = 24 * 60 * 60 * 1000;

// ✅ 複雑なロジックの説明
function calculateWorkingHours(clockIn: Date, clockOut: Date): number {
  // 休憩時間（12:00-13:00）を勤務時間から除外
  // 深夜勤務（22:00-5:00）は割増計算
  const baseHours = (clockOut.getTime() - clockIn.getTime()) / (1000 * 60 * 60);
  // ... 複雑な計算
}
```
