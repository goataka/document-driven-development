# バックエンド設計

本ドキュメントは、NestJS + TypeScriptをベースとしたバックエンド設計の詳細を説明します。

**関連ドキュメント**: [システムアーキテクチャ設計書](../implement/ARCHITECTURE.md)

## 目次

1. [ディレクトリ構成](#ディレクトリ構成)
2. [主要設計パターン](#主要設計パターン)
3. [実装例](#実装例)

---

## ディレクトリ構成

```
backend/
├── src/
│   ├── main.ts                      # エントリーポイント
│   ├── app.module.ts                # ルートモジュール
│   ├── config/                      # 設定ファイル
│   │   ├── database.config.ts
│   │   └── jwt.config.ts
│   ├── common/                      # 共通モジュール
│   │   ├── filters/                 # 例外フィルター
│   │   ├── guards/                  # 認証ガード
│   │   ├── interceptors/            # インターセプター
│   │   ├── decorators/              # カスタムデコレーター
│   │   └── pipes/                   # バリデーションパイプ
│   ├── modules/                     # 機能モジュール
│   │   ├── auth/                    # 認証モジュール
│   │   │   ├── auth.module.ts
│   │   │   ├── auth.controller.ts
│   │   │   ├── auth.service.ts
│   │   │   ├── dto/
│   │   │   └── strategies/
│   │   ├── users/                   # ユーザーモジュール
│   │   │   ├── users.module.ts
│   │   │   ├── users.controller.ts
│   │   │   ├── users.service.ts
│   │   │   ├── entities/
│   │   │   └── dto/
│   │   └── attendance/              # 勤怠モジュール
│   │       ├── attendance.module.ts
│   │       ├── attendance.controller.ts
│   │       ├── attendance.service.ts
│   │       ├── entities/
│   │       └── dto/
│   └── database/                    # データベース関連
│       ├── migrations/              # マイグレーション
│       └── seeds/                   # シードデータ
├── test/                            # テスト
│   ├── unit/
│   └── e2e/
├── package.json
├── tsconfig.json
├── nest-cli.json
└── .env.example
```

---

## 主要設計パターン

### 1. モジュール構成

#### 原則

NestJSのモジュールシステムを活用し、機能ごとに独立したモジュールを作成：
- 各モジュールは必要な機能のみをインポート
- プロバイダーの適切なエクスポート設定
- 疎結合な設計

#### 実装例: AuthModule

```typescript
// src/modules/auth/auth.module.ts
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { PassportModule } from '@nestjs/passport';
import { UsersModule } from '../users/users.module';
import { AuthController } from './auth.controller';
import { AuthService } from './auth.service';
import { JwtStrategy } from './strategies/jwt.strategy';

@Module({
  imports: [
    UsersModule,
    PassportModule,
    JwtModule.register({
      secret: process.env.JWT_SECRET,
      signOptions: { expiresIn: '7d' }
    })
  ],
  controllers: [AuthController],
  providers: [AuthService, JwtStrategy],
  exports: [AuthService]
})
export class AuthModule {}
```

---

### 2. コントローラー設計

#### 原則

- RESTful APIの原則に従ったエンドポイント設計
- Swaggerデコレーターによる自動ドキュメント生成
- ガードによる認証・認可制御
- DTOによるバリデーション

#### 実装例: AttendanceController

```typescript
// src/modules/attendance/attendance.controller.ts
import { Controller, Post, Get, Query, UseGuards } from '@nestjs/common';
import { ApiTags, ApiOperation, ApiResponse, ApiBearerAuth } from '@nestjs/swagger';
import { JwtAuthGuard } from '../auth/guards/jwt-auth.guard';
import { GetUser } from '../auth/decorators/get-user.decorator';
import { AttendanceService } from './attendance.service';
import { User } from '../users/entities/user.entity';
import { GetHistoryDto } from './dto/get-history.dto';
import { Attendance } from './entities/attendance.entity';

@Controller('attendance')
@UseGuards(JwtAuthGuard)
@ApiBearerAuth()
@ApiTags('attendance')
export class AttendanceController {
  constructor(private readonly attendanceService: AttendanceService) {}

  @Post('clock-in')
  @ApiOperation({ summary: '出勤打刻' })
  @ApiResponse({ status: 201, description: '打刻成功' })
  async clockIn(@GetUser() user: User): Promise<Attendance> {
    return this.attendanceService.clockIn(user.id);
  }

  @Post('clock-out')
  @ApiOperation({ summary: '退勤打刻' })
  async clockOut(@GetUser() user: User): Promise<Attendance> {
    return this.attendanceService.clockOut(user.id);
  }

  @Get('history')
  @ApiOperation({ summary: '勤怠履歴取得' })
  async getHistory(
    @GetUser() user: User,
    @Query() query: GetHistoryDto
  ): Promise<Attendance[]> {
    return this.attendanceService.getHistory(user.id, query);
  }
}
```

---

### 3. サービス層

#### 原則

- ビジネスロジックの実装場所
- データベース操作のカプセル化
- トランザクション管理
- エラーハンドリング

#### 実装例: AttendanceService

```typescript
// src/modules/attendance/attendance.service.ts
import { Injectable, BadRequestException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository, Not, IsNull, Between } from 'typeorm';
import { Attendance } from './entities/attendance.entity';
import { GetHistoryDto } from './dto/get-history.dto';

@Injectable()
export class AttendanceService {
  constructor(
    @InjectRepository(Attendance)
    private attendanceRepository: Repository<Attendance>
  ) {}

  async clockIn(userId: string): Promise<Attendance> {
    // 既に出勤打刻済みかチェック
    const existing = await this.attendanceRepository.findOne({
      where: {
        userId,
        clockInTime: Not(IsNull()),
        clockOutTime: IsNull()
      }
    });

    if (existing) {
      throw new BadRequestException('既に出勤打刻済みです');
    }

    const attendance = this.attendanceRepository.create({
      userId,
      clockInTime: new Date()
    });

    return this.attendanceRepository.save(attendance);
  }

  async clockOut(userId: string): Promise<Attendance> {
    const attendance = await this.attendanceRepository.findOne({
      where: {
        userId,
        clockInTime: Not(IsNull()),
        clockOutTime: IsNull()
      }
    });

    if (!attendance) {
      throw new BadRequestException('出勤打刻が見つかりません');
    }

    attendance.clockOutTime = new Date();
    return this.attendanceRepository.save(attendance);
  }

  async getHistory(
    userId: string,
    query: GetHistoryDto
  ): Promise<Attendance[]> {
    const { startDate, endDate } = query;
    
    return this.attendanceRepository.find({
      where: {
        userId,
        clockInTime: Between(startDate, endDate)
      },
      order: { clockInTime: 'DESC' }
    });
  }
}
```

---

### 4. DTO（Data Transfer Object）

#### 原則

- class-validatorによるバリデーション
- class-transformerによる型変換
- Swaggerデコレーターによるドキュメント化

#### 実装例: GetHistoryDto

```typescript
// src/modules/attendance/dto/get-history.dto.ts
import { ApiProperty } from '@nestjs/swagger';
import { IsDateString, IsOptional, IsInt, Min, Max } from 'class-validator';

export class GetHistoryDto {
  @ApiProperty({ description: '開始日', example: '2024-01-01' })
  @IsDateString()
  startDate: string;

  @ApiProperty({ description: '終了日', example: '2024-01-31' })
  @IsDateString()
  endDate: string;

  @ApiProperty({ description: 'ページ番号', example: 1, required: false })
  @IsOptional()
  @IsInt()
  @Min(1)
  page?: number = 1;

  @ApiProperty({ description: '1ページあたりの件数', example: 20, required: false })
  @IsOptional()
  @IsInt()
  @Min(1)
  @Max(100)
  limit?: number = 20;
}
```

---

### 5. エンティティ定義

#### 原則

- TypeORMデコレーターを使用
- リレーションの定義
- インデックスの設定
- 自動タイムスタンプ

#### 実装例: User Entity

```typescript
// src/modules/users/entities/user.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, OneToMany, Index, CreateDateColumn, UpdateDateColumn } from 'typeorm';
import { Attendance } from '../../attendance/entities/attendance.entity';

@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  @Index()
  email: string;

  @Column()
  password: string;

  @Column()
  companyCode: string;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @OneToMany(() => Attendance, (attendance) => attendance.user)
  attendances: Attendance[];
}
```

#### 実装例: Attendance Entity

```typescript
// src/modules/attendance/entities/attendance.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne, JoinColumn, Index, CreateDateColumn, UpdateDateColumn } from 'typeorm';
import { User } from '../../users/entities/user.entity';

@Entity('attendances')
export class Attendance {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  @Index()
  userId: string;

  @ManyToOne(() => User, (user) => user.attendances)
  @JoinColumn({ name: 'userId' })
  user: User;

  @Column({ type: 'timestamp' })
  @Index()
  clockInTime: Date;

  @Column({ type: 'timestamp', nullable: true })
  clockOutTime: Date | null;

  @Column({ type: 'int', nullable: true })
  workDurationMinutes: number | null;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

---

## 実装例

詳細な実装例については、各セクションに記載されています。

**関連ドキュメント**:
- [システムアーキテクチャ設計書](../implement/ARCHITECTURE.md)
- [フロントエンド設計](./FRONTEND.md)
- [データベース設計](./DATABASE.md)
- [API設計](./API.md)
- [セキュリティ](./SECURITY.md)
- [テスト戦略](./TESTING.md)

---

**最終更新日**: 2024年12月17日  
**バージョン**: 1.0.0
