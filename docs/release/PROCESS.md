# リリースプロセス

本ドキュメントは、リリースの手順と自動化を説明します。

**関連ドキュメント**: 
- [リリース管理概要](./README.md)
- [バージョニング戦略](./VERSIONING.md)

## リリース手順

### Step 1: リリースブランチ作成

```bash
git checkout develop
git pull origin develop
git checkout -b release/v1.2.0
npm version minor --no-git-tag-version
git add package.json package-lock.json
git commit -m "chore: bump version to 1.2.0"
git push origin release/v1.2.0
```

### Step 2: リリース候補テスト

```bash
npm run build
npm run test
npm run test:e2e
npm audit
```

### Step 3: マージとタグ付け

```bash
git checkout main
git merge --no-ff release/v1.2.0
git push origin main
git tag -a v1.2.0 -m "Release version 1.2.0"
git push origin v1.2.0
```

---

**最終更新日**: 2024年12月19日  
**バージョン**: 1.0.0
