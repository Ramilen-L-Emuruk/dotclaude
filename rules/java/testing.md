---
paths:
  - "**/*.java"
  - "**/pom.xml"
  - "**/build.gradle"
  - "**/build.gradle.kts"
---
# Java テスト要件

汎用的な TDD ワークフロー・エッジケース・アンチパターン・品質チェックリストは [testing.md](../common/testing.md) を参照。本ファイルは JUnit 5 / Mockito を使った具体的な書き方をまとめる。

## テスト技術スタック

- **JUnit 5** — テストフレームワーク
- **Mockito** — モックフレームワーク

## テスト構造（AAA パターン）

```java
@Test
@DisplayName("フィールドが条件を満たす場合trueを返す")
void isValid_withValidInput_returnsTrue() {
    // Arrange
    String input = "sample";

    // Act
    boolean result = Validator.isValid(input);

    // Assert
    assertTrue(result);
}
```

### テスト命名

英語メソッド名 + `@DisplayName` で日本語説明を付与:

```java
@Test
@DisplayName("入力値がnullの場合空文字を返す")
void convert_withNullInput_returnsEmpty() {}

@Test
@DisplayName("必須項目が未設定の場合に例外をスローする")
void validate_withoutRequiredField_throwsException() {}
```

## テスト実行コマンド

```bash
# 全テスト実行
mvn test

# 単一テストクラスの実行
mvn test -Dtest=ClassNameTest

# 単一テストメソッドの実行
mvn test -Dtest=ClassNameTest#methodName
```
