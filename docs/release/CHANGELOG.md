# チェンジログ管理

本ドキュメントは、チェンジログの管理方法を説明します。

## CHANGELOG.md の構造

```markdown
# Changelog

## [Unreleased]
### Added
- 開発中の新機能

## [1.2.0] - 2024-12-19
### Added
- 勤怠記録の一括エクスポート機能

### Fixed
- 勤怠記録の重複登録問題
```

## コミットメッセージ規約

```bash
# Conventional Commits 形式
<type>(<scope>): <subject>

# 例
feat(auth): add password reset functionality
fix(attendance): correct timezone calculation
```

---

**最終更新日**: 2024年12月19日  
**バージョン**: 1.0.0
