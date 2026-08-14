---
paths:
  - "**/*.java"
  - "**/pom.xml"
  - "**/build.gradle"
  - "**/build.gradle.kts"
---
# Java フック

> 本ファイルは [common/hooks.md](../common/hooks.md) を Java 固有の内容で拡張する。

## PostToolUse フック

`~/.claude/settings.json` で設定:

- **google-java-format**: 編集後に `.java` ファイルを自動フォーマット
- **checkstyle**: Java ファイル編集後にスタイルチェックを実行
- **./mvnw compile** または **./gradlew compileJava**: 変更後のコンパイル検証
