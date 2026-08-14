---
paths:
  - "**/*.java"
---
# Java コードレビュー基準

汎用的なレビュータイミング・重要度レベル・承認基準は [code-review.md](../common/code-review.md) を参照。本ファイルは Java/Spring Boot 固有のチェックリストをまとめる。

## レビューチェックリスト

### Java スタイル（reviewer エージェント担当）

- [ ] クラス／メソッド宣言直後に不要な空行がないこと
- [ ] メソッド末尾の閉じ括弧前に不要な空行がないこと
- [ ] 空の `@param`、`@return`、`@throws` タグに説明が記載されていること
- [ ] クラス JavaDoc が適切に記述されていること
- [ ] 定数フィールドに `final` 修飾子が付いていること
- [ ] 未使用の import が除去されていること
- [ ] コメントアウトされたデッドコードが残っていないこと
- [ ] レガシー API（`Calendar`/`Date`）が `java.time` に置き換えられていること

### Java/Spring Boot 専門（java-reviewer エージェント担当）

- [ ] JPA エンティティの関連マッピングが適切であること（N+1 問題の回避）
- [ ] `@Transactional` のスコープが適切であること（readOnly の使い分け）
- [ ] Spring Bean のスコープとライフサイクルが正しいこと
- [ ] 並行処理の安全性（スレッドセーフ、デッドロック回避）
- [ ] Controller-Service-Repository の責務分離が適切であること

### JavaDoc の正確性（comment-analyzer エージェント担当）

- [ ] JavaDoc の `@param`・`@return`・`@throws` が実装と一致していること

## エージェントの使い分け

| エージェント | 担当範囲 | 使用条件 |
|-------------|---------|---------|
| **java-reviewer** | Java/Spring Boot 専門: JPA、アーキテクチャ、並行処理、Bean スコープ | Spring Boot / JPA / 並行処理の変更時 |
