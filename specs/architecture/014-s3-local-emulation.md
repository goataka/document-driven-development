# ADR-014: S3ローカルエミュレーション戦略

## ステータス

提案中（Proposed）

## 背景

ローカル開発環境やCI環境において、AWS S3のエミュレーションが必要になる場合があります。ファイルアップロード、静的ファイルホスティング、バックアップなどの機能をローカルで開発・テストするための戦略を検討します。

### S3のユースケース

1. **ファイルアップロード**: ユーザープロファイル画像、ドキュメント
2. **静的ファイルホスティング**: フロントエンドアセット（将来的な拡張）
3. **バックアップストレージ**: データベースバックアップ
4. **ログ保存**: アプリケーションログの長期保存

## 決定事項

**オプション**: S3の使用が必要な場合のみ、LocalStackを使用したローカルエミュレーションを導入する。初期実装では不要。

### S3エミュレーションを導入すべきケース

1. **ファイルアップロード機能**: アプリケーションでファイル管理が必要
2. **CI/CDテスト**: S3統合テストが必要
3. **ローカル開発**: AWS アカウントなしで開発したい
4. **コスト削減**: 開発環境のS3コスト削減

### S3エミュレーションが不要なケース

1. **S3未使用**: アプリケーションでS3を使用しない
2. **シンプルな構成**: ファイル管理が不要
3. **直接AWS使用**: 開発環境でも実際のS3を使用

## 結果

### LocalStack の選択肢

1. **LocalStack Community（無料）**: 基本的なS3機能
2. **LocalStack Pro（有料）**: 高度な機能、エンタープライズサポート
3. **MinIO**: S3互換のオープンソースストレージ
4. **実際のS3**: AWS S3を直接使用

### メリット（LocalStack Community）

- **完全無料**: Community版で十分
- **AWS SDK互換**: コード変更なし
- **Docker対応**: 簡単なセットアップ
- **複数サービス**: DynamoDB、S3、SNS等を同時エミュレーション
- **オフライン開発**: インターネット接続不要

### デメリット

- **制限あり**: Pro版の機能は使用不可
- **完全互換ではない**: 一部の高度な機能は未サポート
- **パフォーマンス**: 実際のS3より遅い場合がある
- **バグ**: エミュレーション特有の問題が発生する可能性

## 実装例

### 1. LocalStack セットアップ（Docker Compose）

```yaml
# docker-compose.yml
version: '3.8'

services:
  localstack:
    image: localstack/localstack:latest
    container_name: localstack
    ports:
      - "4566:4566" # LocalStack Gateway
      - "4510-4559:4510-4559" # External services port range
    environment:
      - SERVICES=s3,dynamodb
      - DEBUG=1
      - DATA_DIR=/tmp/localstack/data
      - DOCKER_HOST=unix:///var/run/docker.sock
    volumes:
      - "./localstack-data:/tmp/localstack"
      - "/var/run/docker.sock:/var/run/docker.sock"
    networks:
      - app-network

  backend:
    build: ./backend
    depends_on:
      - localstack
    environment:
      - AWS_ENDPOINT=http://localstack:4566
      - AWS_REGION=ap-northeast-1
      - AWS_ACCESS_KEY_ID=test
      - AWS_SECRET_ACCESS_KEY=test
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

### 2. S3バケット初期化スクリプト

```bash
#!/bin/bash
# scripts/init-localstack.sh

# LocalStackの起動を待つ
echo "Waiting for LocalStack..."
until aws --endpoint-url=http://localhost:4566 s3 ls > /dev/null 2>&1; do
  sleep 1
done

echo "Creating S3 buckets..."
aws --endpoint-url=http://localhost:4566 s3 mb s3://user-uploads
aws --endpoint-url=http://localhost:4566 s3 mb s3://app-backups

echo "LocalStack initialized successfully!"
```

### 3. アプリケーションコード（環境別切り替え）

```typescript
import { S3Client } from '@aws-sdk/client-s3';
import { PutObjectCommand, GetObjectCommand } from '@aws-sdk/client-s3';

// 環境に応じてS3クライアントを作成
function createS3Client(): S3Client {
  const config: any = {
    region: process.env.AWS_REGION || 'ap-northeast-1',
  };

  // ローカル環境ではLocalStackを使用
  if (process.env.NODE_ENV === 'development' && process.env.AWS_ENDPOINT) {
    config.endpoint = process.env.AWS_ENDPOINT; // http://localhost:4566
    config.credentials = {
      accessKeyId: 'test',
      secretAccessKey: 'test',
    };
    config.forcePathStyle = true; // LocalStackではPathStyle必須
  }

  return new S3Client(config);
}

const s3Client = createS3Client();

// ファイルアップロード
export async function uploadFile(
  bucketName: string,
  key: string,
  body: Buffer,
  contentType: string,
): Promise<void> {
  await s3Client.send(
    new PutObjectCommand({
      Bucket: bucketName,
      Key: key,
      Body: body,
      ContentType: contentType,
    })
  );
}

// ファイルダウンロード
export async function downloadFile(
  bucketName: string,
  key: string,
): Promise<Buffer> {
  const response = await s3Client.send(
    new GetObjectCommand({
      Bucket: bucketName,
      Key: key,
    })
  );
  
  return Buffer.from(await response.Body.transformToByteArray());
}
```

### 4. NestJS統合

```typescript
// s3.service.ts
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';

@Injectable()
export class S3Service {
  private readonly s3Client: S3Client;
  private readonly bucketName: string;

  constructor(private readonly configService: ConfigService) {
    const config: any = {
      region: this.configService.get('AWS_REGION'),
    };

    // LocalStack対応
    const endpoint = this.configService.get('AWS_ENDPOINT');
    if (endpoint) {
      config.endpoint = endpoint;
      config.credentials = {
        accessKeyId: 'test',
        secretAccessKey: 'test',
      };
      config.forcePathStyle = true;
    }

    this.s3Client = new S3Client(config);
    this.bucketName = this.configService.get('S3_BUCKET_NAME');
  }

  async uploadUserAvatar(userId: string, file: Buffer): Promise<string> {
    const key = `avatars/${userId}.jpg`;
    
    await this.s3Client.send(
      new PutObjectCommand({
        Bucket: this.bucketName,
        Key: key,
        Body: file,
        ContentType: 'image/jpeg',
      })
    );

    return key;
  }
}
```

### 5. テストコード

```typescript
// s3.service.spec.ts
import { Test } from '@nestjs/testing';
import { ConfigService } from '@nestjs/config';
import { S3Service } from './s3.service';

describe('S3Service', () => {
  let service: S3Service;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        S3Service,
        {
          provide: ConfigService,
          useValue: {
            get: jest.fn((key: string) => {
              const config = {
                AWS_REGION: 'ap-northeast-1',
                AWS_ENDPOINT: 'http://localhost:4566', // LocalStack
                S3_BUCKET_NAME: 'test-bucket',
              };
              return config[key];
            }),
          },
        },
      ],
    }).compile();

    service = module.get<S3Service>(S3Service);
  });

  it('should upload file to S3', async () => {
    const userId = 'test-user-123';
    const fileBuffer = Buffer.from('test image data');

    const key = await service.uploadUserAvatar(userId, fileBuffer);
    
    expect(key).toBe('avatars/test-user-123.jpg');
  });
});
```

## 環境別構成

### ローカル環境
```env
NODE_ENV=development
AWS_REGION=ap-northeast-1
AWS_ENDPOINT=http://localhost:4566
AWS_ACCESS_KEY_ID=test
AWS_SECRET_ACCESS_KEY=test
S3_BUCKET_NAME=user-uploads
```

### CI環境
```yaml
# .github/workflows/test.yml
services:
  localstack:
    image: localstack/localstack
    env:
      SERVICES: s3,dynamodb
    ports:
      - 4566:4566
```

### 本番環境
```env
NODE_ENV=production
AWS_REGION=ap-northeast-1
# AWS_ENDPOINT は設定しない（実際のS3を使用）
S3_BUCKET_NAME=prod-user-uploads
```

## 代替案

### 1. MinIO

```yaml
# docker-compose.yml
services:
  minio:
    image: minio/minio
    ports:
      - "9000:9000"
      - "9001:9001"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    command: server /data --console-address ":9001"
```

**メリット**: S3完全互換、高パフォーマンス  
**デメリット**: DynamoDB等はエミュレートできない

### 2. 実際のS3を使用

```typescript
// 開発環境でも実際のS3を使用
// AWS認証情報を設定
const s3Client = new S3Client({
  region: 'ap-northeast-1',
  // ~/.aws/credentials から自動読み込み
});
```

**メリット**: 完全に本番同等  
**デメリット**: コスト、AWS認証情報管理

### 3. ローカルファイルシステム

```typescript
// S3の代わりにローカルファイルシステムを使用
import fs from 'fs/promises';
import path from 'path';

export async function uploadFile(key: string, body: Buffer): Promise<void> {
  const filePath = path.join('./uploads', key);
  await fs.mkdir(path.dirname(filePath), { recursive: true });
  await fs.writeFile(filePath, body);
}
```

**メリット**: 最もシンプル  
**デメリット**: S3の機能（ACL、バージョニング等）が使えない

## コスト比較

| 構成 | セットアップコスト | 運用コスト（開発） | 本番互換性 |
|------|------------------|------------------|-----------|
| LocalStack | 低 | $0 | 中 |
| MinIO | 中 | $0 | 高 |
| 実際のS3 | 低 | $1-5/月 | 完全 |
| ローカルFS | 最低 | $0 | 低 |

## 推奨構成

### 現在（S3未使用）
- **エミュレーション**: 不要
- **将来の拡張**: LocalStack準備

### Phase 1（ファイルアップロード実装時）
- **ローカル**: LocalStack
- **CI**: LocalStack
- **本番**: 実際のS3

### Phase 2（本番運用）
- **ローカル**: LocalStack（開発用データ）
- **ステージング**: 実際のS3（分離バケット）
- **本番**: 実際のS3

## 学習リソース

- [LocalStack Documentation](https://docs.localstack.cloud/)
- [AWS SDK for JavaScript v3 - S3](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/clients/client-s3/)
- [MinIO Documentation](https://min.io/docs/minio/linux/index.html)

## 関連ドキュメント

- [バックエンドデプロイ方針](../../docs/deploy/BACKEND.md)
- [ビルド戦略](../../docs/build/BACKEND.md)
- [テスト戦略](../../docs/test/INTEGRATION.md)

## 更新履歴

- 2024-12-21: 初版作成
