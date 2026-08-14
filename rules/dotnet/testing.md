---
paths:
  - "**/*.cs"
  - "**/*.csproj"
  - "**/*.sln"
---
# .NET テスト要件

汎用的な TDD ワークフロー・エッジケース・アンチパターン・品質チェックリストは [testing.md](../common/testing.md) を参照。本ファイルは xUnit を使った具体的な書き方をまとめる。

## テスト技術スタック

- **xUnit** — テストフレームワーク
- **coverlet** — カバレッジ計測（`--collect:"XPlat Code Coverage"`）
- モックが必要な場合は **Moq** または **NSubstitute**。ただし導入済みかはプロジェクトごとに `.csproj` で確認する（未導入の場合、モックを前提としたテストは書けない）

## テスト構造（AAA パターン）

```csharp
[Fact(DisplayName = "入力が条件を満たす場合 true を返す")]
public void IsValid_WithValidInput_ReturnsTrue()
{
    // Arrange
    var input = "sample";

    // Act
    var result = Validator.IsValid(input);

    // Assert
    Assert.True(result);
}
```

### パラメータ化テスト

同じ検証を複数の入力で回す場合は `[Theory]` + `[InlineData]` を使う:

```csharp
[Theory(DisplayName = "空白のみの入力は不正と判定される")]
[InlineData("")]
[InlineData(" ")]
[InlineData("\t")]
public void IsValid_WithBlankInput_ReturnsFalse(string input)
{
    Assert.False(Validator.IsValid(input));
}
```

### テスト命名

`メソッド名_条件_期待結果` の PascalCase に、`DisplayName` で日本語説明を付与する:

```csharp
[Fact(DisplayName = "入力値が null の場合に空文字を返す")]
public void Convert_WithNullInput_ReturnsEmpty() { }

[Fact(DisplayName = "必須項目が未設定の場合に例外をスローする")]
public void Validate_WithoutRequiredField_ThrowsException() { }
```

## テスト対象の切り分け

unsafe コード・P/Invoke・GPU リソースなど実行環境に依存する層は、ユニットテストの対象外とすることが多い。その場合は純ロジック（状態機械・キュー・計算処理）を別クラスへ切り出してテスト可能にし、**何をテスト対象外としたかをプロジェクトのドキュメントに明記する**（暗黙にカバレッジが低いだけの状態と区別できなくなるため）。

## テスト実行コマンド

```bash
# 全テスト実行
dotnet test <テストプロジェクトのパス>

# 単一テストクラスの実行
dotnet test <テストプロジェクトのパス> --filter "FullyQualifiedName~ClassName"

# 単一テストメソッドの実行
dotnet test <テストプロジェクトのパス> --filter "FullyQualifiedName~ClassName.MethodName"

# カバレッジ付き実行
dotnet test <テストプロジェクトのパス> --collect:"XPlat Code Coverage"
```
