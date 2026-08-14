---
paths:
  - "**/*.cs"
  - "**/*.csproj"
  - "**/*.sln"
---
# .NET パフォーマンスの考慮事項

## 実装レベルの最適化

- ループ内の文字列連結は `StringBuilder` を使用
- 配列・文字列の部分コピーを避ける: `Span<T>` / `ReadOnlySpan<T>` / `Memory<T>`
- 大きなバッファを繰り返し確保する箇所は `ArrayPool<T>.Shared` で使い回す
- ホットパス（毎フレーム・毎リクエスト実行される経路）の LINQ はイテレータとクロージャのアロケーションを生む。素の `for` / `foreach` を検討する
- 構造体を `object` やインターフェース経由で扱うと boxing が発生する
- 不変の計算結果は `static readonly` フィールドにキャッシュ
- `Enum.GetValues()` は呼び出しごとに新しい配列を生成するため、キャッシュして再利用する
- `struct` は値渡しでコピーが発生する。大きい構造体は `in` / `ref` で渡す

## .NET ビルドコマンド

```bash
dotnet build                                    # コンパイルエラーの確認
dotnet build -c Release                         # リリースビルド
dotnet test <テストプロジェクトのパス>            # テスト実行
dotnet publish <csproj> -c Release -o <出力先>   # 発行
```

ビルド失敗時の対応は [performance.md](../common/performance.md) の「ビルドトラブルシューティング」を参照。
