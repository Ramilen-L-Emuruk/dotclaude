---
paths:
  - "**/*.cs"
  - "**/*.csproj"
  - "**/*.sln"
  - "**/*.xaml"
---
# .NET エージェント構成

汎用的なエージェント一覧・自動呼び出しルール・並列実行パターンの全体像は [agents.md](../common/agents.md) を参照。本ファイルはそこから切り出した C#/.NET 固有の運用をまとめる。

## .NET 固有の専用エージェントは用意していない

Java/Spring Boot には `reviewer` / `java-reviewer` / `java-build-resolver` という言語固有の専用エージェントがあるが、C#/.NET に対応するものは置いていない。汎用エージェント（`code-reviewer`・`build-error-resolver`・`code-architect` 等）を使い、[code-review.md](./code-review.md) のチェックリストを**プロンプトで明示的に渡して**補う。

## ビルドエラー時のトリガー

| トリガー条件 | 呼び出すエージェント |
|-------------|-------------------|
| `dotnet build` 失敗 | **build-error-resolver** |
| NuGet の復元失敗・パッケージバージョン競合 | **build-error-resolver**（`dotnet restore` の出力全文をプロンプトに含める） |
| `dotnet test` 失敗（ビルドは成功） | **build-error-resolver** |

## レビュー時のトリガー

| トリガー条件 | 呼び出すエージェント |
|-------------|-------------------|
| `async` / `await`・スレッド間通信の変更 | **code-reviewer**（「並行性・UI スレッド・デッドロック」を観点として明示する） |
| unsafe ブロック・P/Invoke・ネイティブ相互運用の変更 | **code-reviewer** + **silent-failure-hunter** の並列（解放漏れ・戻り値の未検査を検出する） |
| `IDisposable` を実装・保持する型の変更 | **code-reviewer**（「リソース解放・例外経路」を観点として明示する） |
| XML ドキュメントコメントの追加・変更 | **comment-analyzer** |

## 拡張レビューの並列パターン

```markdown
# 拡張レビュー（非同期処理・ネイティブ相互運用の変更時）
並列起動:
1. **code-reviewer**: コード品質（rules/dotnet/code-review.md の観点を明示して渡す）
2. **silent-failure-hunter**: 握り潰された例外・戻り値の未検査・解放漏れ
3. **security-reviewer**: セキュリティ分析（外部入力・ファイル操作を含む場合）
```
