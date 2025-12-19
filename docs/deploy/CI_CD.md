# CI/CD パイプライン

本ドキュメントは、CI/CDデプロイ設定を説明します。

**関連ドキュメント**: 
- [デプロイ戦略概要](./README.md)
- [ビルド戦略](../build/)

## GitHub Actions デプロイ

```yaml
name: Deploy
on:
  push:
    tags: ['v*']
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy
        run: npm run deploy
```

---

**最終更新日**: 2024年12月19日  
**バージョン**: 1.0.0
