# 監視戦略

本ドキュメントは、システムの監視戦略、ログ管理、アラート設定を説明します。
**すべて無料のツールを使用しています。**

**関連ドキュメント**: 
- [システムアーキテクチャ設計書](../implement/)
- [運用管理](../operate/)
- [デプロイ戦略](../deploy/)

## 目次

1. [概要](#概要)
2. [監視の種類](#監視の種類)
3. [ログ管理](#ログ管理)
4. [メトリクス収集](#メトリクス収集)
5. [アラート設定](#アラート設定)
6. [パフォーマンス監視](#パフォーマンス監視)
7. [エラー追跡](#エラー追跡)
8. [ダッシュボード](#ダッシュボード)

---

## 概要

本システムの監視戦略は、システムの健全性、パフォーマンス、セキュリティを継続的に監視し、問題を早期に検知して対応することを目的とします。

### 監視の目的

- **可用性監視**: システムが正常に稼働しているか
- **パフォーマンス監視**: レスポンスタイムやスループット
- **エラー監視**: アプリケーションエラーや例外
- **セキュリティ監視**: 不審なアクティビティや攻撃
- **リソース監視**: CPU、メモリ、ディスク使用率

### 監視ツールスタック（無料）

| カテゴリ | ツール | 用途 |
|---------|-------|------|
| **ログ管理** | Winston / Pino | アプリケーションログ |
| **メトリクス** | Prometheus | メトリクス収集 |
| **可視化** | Grafana | ダッシュボード |
| **エラー追跡** | Sentry（無料プラン） | エラー監視 |
| **アップタイム監視** | UptimeRobot（無料プラン） | サービス死活監視 |
| **APM** | OpenTelemetry | トレーシング |

---

## 監視の種類

### 1. ヘルスチェック監視

#### アプリケーションヘルスチェック

```typescript
// backend/src/health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import { HealthCheck, HealthCheckService, TypeOrmHealthIndicator } from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private db: TypeOrmHealthIndicator,
  ) {}

  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      // データベース接続確認
      () => this.db.pingCheck('database'),
      
      // メモリ使用量確認
      () => ({
        memory: {
          status: process.memoryUsage().heapUsed < 500 * 1024 * 1024 ? 'up' : 'down',
          heapUsed: process.memoryUsage().heapUsed,
        },
      }),
      
      // ディスク使用量確認（オプション）
    ]);
  }

  @Get('ready')
  @HealthCheck()
  readiness() {
    // Readiness probe: アプリケーションがトラフィックを受け付ける準備ができているか
    return this.health.check([
      () => this.db.pingCheck('database'),
    ]);
  }

  @Get('live')
  @HealthCheck()
  liveness() {
    // Liveness probe: アプリケーションが生きているか
    return { status: 'ok', timestamp: new Date().toISOString() };
  }
}
```

### 2. アップタイム監視

#### UptimeRobot 設定

```markdown
# UptimeRobot 監視設定

1. https://uptimerobot.com/ でアカウント作成（無料）

2. モニター追加:
   - Monitor Type: HTTP(s)
   - URL: https://api.example.com/health
   - Monitoring Interval: 5 minutes
   - Alert Contacts: メール、Slack

3. アラート条件:
   - ダウンタイム: 即座に通知
   - レスポンスタイム: 2秒以上で警告
```

### 3. リソース監視

#### システムリソース

```bash
# CPU使用率
mpstat 1 5

# メモリ使用率
free -m

# ディスク使用率
df -h

# ネットワーク統計
netstat -s

# Node.jsプロセス監視
pm2 monit
```

---

## ログ管理

### ログレベル

| レベル | 用途 | 例 |
|-------|------|-----|
| **ERROR** | エラー、例外 | データベース接続エラー |
| **WARN** | 警告 | 非推奨APIの使用 |
| **INFO** | 情報 | ユーザーログイン |
| **DEBUG** | デバッグ | 関数の実行トレース |
| **TRACE** | 詳細トレース | 変数の値 |

### ロガー設定

#### バックエンド（Winston）

```typescript
// backend/src/common/logger/logger.config.ts
import * as winston from 'winston';
import * as DailyRotateFile from 'winston-daily-rotate-file';

export const loggerConfig = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  transports: [
    // コンソール出力
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple()
      )
    }),
    
    // エラーログファイル
    new DailyRotateFile({
      filename: 'logs/error-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      level: 'error',
      maxFiles: '30d',
      maxSize: '20m'
    }),
    
    // 全ログファイル
    new DailyRotateFile({
      filename: 'logs/application-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      maxFiles: '30d',
      maxSize: '20m'
    })
  ]
});
```

#### フロントエンド（カスタムロガー）

```typescript
// frontend/src/utils/logger.ts
type LogLevel = 'debug' | 'info' | 'warn' | 'error';

class Logger {
  private isDevelopment = import.meta.env.DEV;

  private log(level: LogLevel, message: string, meta?: any) {
    if (!this.isDevelopment && level === 'debug') {
      return; // 本番環境ではdebugログを出力しない
    }

    const timestamp = new Date().toISOString();
    const logMessage = `[${timestamp}] [${level.toUpperCase()}] ${message}`;

    switch (level) {
      case 'debug':
        console.debug(logMessage, meta);
        break;
      case 'info':
        console.info(logMessage, meta);
        break;
      case 'warn':
        console.warn(logMessage, meta);
        break;
      case 'error':
        console.error(logMessage, meta);
        // Sentryにエラーを送信
        if (window.Sentry) {
          window.Sentry.captureException(new Error(message), { extra: meta });
        }
        break;
    }
  }

  debug(message: string, meta?: any) {
    this.log('debug', message, meta);
  }

  info(message: string, meta?: any) {
    this.log('info', message, meta);
  }

  warn(message: string, meta?: any) {
    this.log('warn', message, meta);
  }

  error(message: string, meta?: any) {
    this.log('error', message, meta);
  }
}

export const logger = new Logger();
```

### ログローテーション

```bash
# /etc/logrotate.d/attendance-app
/var/log/attendance/*.log {
    daily
    rotate 30
    compress
    delaycompress
    notifempty
    create 0640 www-data www-data
    sharedscripts
    postrotate
        systemctl reload attendance-backend
    endscript
}
```

---

## メトリクス収集

### Prometheus 設定

#### メトリクスエンドポイント

```typescript
// backend/src/metrics/metrics.controller.ts
import { Controller, Get, Header } from '@nestjs/common';
import { register, Counter, Histogram } from 'prom-client';

// カウンター: HTTPリクエスト数
export const httpRequestCounter = new Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

// ヒストグラム: レスポンスタイム
export const httpRequestDuration = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.5, 1, 2, 5]
});

@Controller('metrics')
export class MetricsController {
  @Get()
  @Header('Content-Type', register.contentType)
  async metrics() {
    return register.metrics();
  }
}
```

#### メトリクスミドルウェア

```typescript
// backend/src/common/middleware/metrics.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';
import { httpRequestCounter, httpRequestDuration } from '../metrics';

@Injectable()
export class MetricsMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const start = Date.now();

    res.on('finish', () => {
      const duration = (Date.now() - start) / 1000;
      const labels = {
        method: req.method,
        route: req.route?.path || req.path,
        status_code: res.statusCode
      };

      httpRequestCounter.inc(labels);
      httpRequestDuration.observe(labels, duration);
    });

    next();
  }
}
```

#### Prometheus設定ファイル

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'attendance-backend'
    static_configs:
      - targets: ['localhost:3000']
    metrics_path: '/metrics'
    
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['localhost:9100']
```

### カスタムメトリクス

```typescript
// backend/src/attendance/attendance.service.ts
import { Gauge } from 'prom-client';

const activeUsersGauge = new Gauge({
  name: 'active_users_count',
  help: 'Number of currently active users'
});

const attendanceRecordsGauge = new Gauge({
  name: 'attendance_records_today',
  help: 'Number of attendance records created today'
});

@Injectable()
export class AttendanceService {
  async updateMetrics() {
    // アクティブユーザー数を更新
    const activeUsers = await this.getActiveUsersCount();
    activeUsersGauge.set(activeUsers);

    // 本日の勤怠記録数を更新
    const todayRecords = await this.getTodayRecordsCount();
    attendanceRecordsGauge.set(todayRecords);
  }
}
```

---

## アラート設定

### Prometheus Alertmanager

#### アラートルール

```yaml
# alerts.yml
groups:
  - name: attendance_alerts
    interval: 30s
    rules:
      # APIレスポンスタイムが5秒を超えた場合
      - alert: HighResponseTime
        expr: http_request_duration_seconds{quantile="0.95"} > 5
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High API response time"
          description: "95th percentile response time is {{ $value }}s"

      # エラーレートが5%を超えた場合
      - alert: HighErrorRate
        expr: rate(http_requests_total{status_code=~"5.."}[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | humanizePercentage }}"

      # データベース接続エラー
      - alert: DatabaseDown
        expr: up{job="postgres"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Database is down"
          description: "PostgreSQL database is not responding"

      # メモリ使用率が90%を超えた場合
      - alert: HighMemoryUsage
        expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes > 0.9
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage"
          description: "Memory usage is {{ $value | humanizePercentage }}"

      # ディスク使用率が85%を超えた場合
      - alert: HighDiskUsage
        expr: (node_filesystem_size_bytes - node_filesystem_free_bytes) / node_filesystem_size_bytes > 0.85
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High disk usage"
          description: "Disk usage is {{ $value | humanizePercentage }}"
```

#### Alertmanager設定

```yaml
# alertmanager.yml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname', 'severity']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  receiver: 'default'
  routes:
    - match:
        severity: critical
      receiver: 'critical-alerts'
    - match:
        severity: warning
      receiver: 'warning-alerts'

receivers:
  - name: 'default'
    email_configs:
      - to: 'ops-team@example.com'
        from: 'alertmanager@example.com'
        smarthost: 'smtp.example.com:587'
        auth_username: 'alertmanager@example.com'
        auth_password: 'password'

  - name: 'critical-alerts'
    email_configs:
      - to: 'on-call@example.com'
        send_resolved: true
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK'
        channel: '#alerts-critical'
        title: 'Critical Alert'

  - name: 'warning-alerts'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK'
        channel: '#alerts-warning'
        title: 'Warning Alert'
```

---

## パフォーマンス監視

### レスポンスタイム監視

```typescript
// backend/src/common/interceptors/performance.interceptor.ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class PerformanceInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, url } = request;
    const start = Date.now();

    return next.handle().pipe(
      tap(() => {
        const duration = Date.now() - start;
        
        // 閾値を超えた場合は警告
        if (duration > 2000) {
          console.warn(`Slow request: ${method} ${url} took ${duration}ms`);
        }
        
        // メトリクスに記録
        httpRequestDuration.observe(
          { method, route: url, status_code: context.switchToHttp().getResponse().statusCode },
          duration / 1000
        );
      })
    );
  }
}
```

### データベースクエリ監視

```typescript
// backend/src/config/database.config.ts
import { TypeOrmModuleOptions } from '@nestjs/typeorm';

export const databaseConfig: TypeOrmModuleOptions = {
  type: 'postgres',
  url: process.env.DATABASE_URL,
  logging: ['error', 'warn', 'schema'],
  logger: 'advanced-console',
  
  // スロークエリロギング
  maxQueryExecutionTime: 1000, // 1秒以上のクエリを記録
};
```

---

## エラー追跡

### Sentry 設定

#### バックエンド

```typescript
// backend/src/main.ts
import * as Sentry from '@sentry/node';

async function bootstrap() {
  Sentry.init({
    dsn: process.env.SENTRY_DSN,
    environment: process.env.NODE_ENV,
    tracesSampleRate: 0.1, // 10%のトランザクションをトレース
  });

  const app = await NestFactory.create(AppModule);
  
  // Sentryエラーハンドラー
  app.use(Sentry.Handlers.requestHandler());
  app.use(Sentry.Handlers.errorHandler());
  
  await app.listen(3000);
}
```

#### フロントエンド

```typescript
// frontend/src/main.tsx
import * as Sentry from '@sentry/react';

Sentry.init({
  dsn: import.meta.env.VITE_SENTRY_DSN,
  environment: import.meta.env.MODE,
  integrations: [
    new Sentry.BrowserTracing(),
    new Sentry.Replay()
  ],
  tracesSampleRate: 0.1,
  replaysSessionSampleRate: 0.1,
  replaysOnErrorSampleRate: 1.0,
});
```

---

## ダッシュボード

### Grafana ダッシュボード

#### システム概要ダッシュボード

```json
{
  "dashboard": {
    "title": "Attendance System Overview",
    "panels": [
      {
        "title": "Request Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])"
          }
        ]
      },
      {
        "title": "Response Time (95th percentile)",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, http_request_duration_seconds_bucket)"
          }
        ]
      },
      {
        "title": "Error Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total{status_code=~\"5..\"}[5m])"
          }
        ]
      },
      {
        "title": "Active Users",
        "targets": [
          {
            "expr": "active_users_count"
          }
        ]
      }
    ]
  }
}
```

---

## ベストプラクティス

1. **包括的な監視**: すべてのレイヤーを監視
2. **適切なアラート**: 重要度に応じたアラート設定
3. **ログの構造化**: JSON形式での構造化ログ
4. **メトリクスの標準化**: 一貫したメトリクス命名規則
5. **ダッシュボードの可視化**: 重要な情報を一目で確認
6. **ログの保持期間**: 適切なログ保持ポリシー
7. **セキュリティログ**: セキュリティイベントの監視
8. **パフォーマンス追跡**: 継続的なパフォーマンス監視
9. **エラー追跡**: すべてのエラーを記録と分析
10. **定期的なレビュー**: 監視設定の定期的な見直し

---

## 関連ドキュメント

- [システムアーキテクチャ設計書](../implement/)
- [運用管理](../operate/)
- [デプロイ戦略](../deploy/)
- [パフォーマンステスト](../test/PERFORMANCE.md)

---

**最終更新日**: 2024年12月19日  
**バージョン**: 1.0.0  
**ドキュメント管理者**: 開発チーム
