---
paths:
  - "**/*.java"
---
# Java/Spring セキュリティガイドライン

汎用的な必須セキュリティチェック・シークレット管理・対応プロトコルは [security.md](../common/security.md) を参照。本ファイルは Java/Spring 固有の対策をまとめる。

## Java/Spring 固有のセキュリティ

- `PreparedStatement` を使用（文字列連結で SQL を組み立てない）
- Spring Security のフィルターチェーンを適切に設定
- `@Validated` / Bean Validation でリクエストパラメータを検証
- セッション管理: `HttpSession` の適切なライフサイクル管理
