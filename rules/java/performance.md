---
paths:
  - "**/*.java"
  - "**/pom.xml"
  - "**/build.gradle"
  - "**/build.gradle.kts"
---
# Java パフォーマンスの考慮事項

## 実装レベルの最適化

- 不変の計算結果は `static final` フィールドにキャッシュ
- `enum.values()` は毎回配列をコピーするため、キャッシュして再利用
- `String` の連結が多い場合は `StringBuilder` を使用
- `Charset.forName()` の結果はフィールドにキャッシュ
- Stream API のオーバーヘッドが問題になる場合は従来のループを使用

## Maven ビルドコマンド

```bash
mvn clean compile          # コンパイルエラーの確認
mvn test                   # テスト実行
mvn package                # パッケージング
mvn install                # ローカルリポジトリにインストール
```

ビルド失敗時の対応は [performance.md](../common/performance.md) の「ビルドトラブルシューティング」を参照。
