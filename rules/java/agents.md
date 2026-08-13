# Java/Spring Boot エージェント構成

汎用的なエージェント一覧・自動呼び出しルール・並列実行パターンの全体像は [agents.md](../common/agents.md) を参照。本ファイルはそこから切り出した Java/Spring Boot 固有の詳細をまとめる。

## Java/Spring Boot 固有のエージェント

| エージェント | モデル | 役割 | 使用タイミング |
|-------------|--------|------|---------------|
| reviewer | default | Java コードスタイルレビュー | フォーマット・JavaDoc・コード規約の確認 |
| java-reviewer | sonnet | Java/Spring Boot 専門レビュー | JPA・アーキテクチャ・並行処理の検証 |
| java-build-resolver | sonnet | Java/Maven/Gradle ビルドエラー修正 | Java ビルド失敗時 |

## ビルドエラー時のトリガー

| トリガー条件 | 呼び出すエージェント |
|-------------|-------------------|
| `mvn compile` 失敗 | **build-error-resolver** |
| Maven 依存関係の解決失敗・プラグインエラー | **java-build-resolver** |
| `mvn test` 失敗（コンパイルは成功） | **build-error-resolver** |

## レビュー時のトリガー

| トリガー条件 | 呼び出すエージェント |
|-------------|-------------------|
| Spring Boot / JPA / 並行処理のコード変更 | **java-reviewer**（`reviewer` + `code-reviewer` に追加） |

## 拡張レビューの並列パターン

```markdown
# 拡張レビュー（Spring Boot / JPA / 並行処理関連の変更時）
並列起動:
1. **reviewer**: Java スタイル
2. **code-reviewer**: コード品質
3. **java-reviewer**: JPA・アーキテクチャ・並行処理
4. **security-reviewer**: セキュリティ分析
```

## ビルドエラー時のエスカレーション

```markdown
# Step 1: 汎用ビルドエラー解決
**build-error-resolver** を起動

# Step 2: Java/Maven 固有の問題が判明した場合
**java-build-resolver** にエスカレーション
```

## エージェント選択の判断基準

| 状況 | 判断 |
|------|------|
| reviewer と java-reviewer のどちらを使うか | **両方**並列で使う。reviewer はスタイル、java-reviewer は Spring Boot 固有の問題を検出 |
| build-error-resolver と java-build-resolver のどちらを使うか | まず **build-error-resolver**。Maven/Gradle 固有の問題なら **java-build-resolver** にエスカレーション |
