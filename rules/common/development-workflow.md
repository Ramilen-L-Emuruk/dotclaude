# 開発ワークフロー

> 本ファイルは [git-workflow.md](./git-workflow.md) を拡張し、git 操作前の開発プロセス全体を定義する。

## 機能実装ワークフロー

0. **調査・再利用**（実装前に必須）
   - **code-explorer** エージェントでコードベースを調査（既存の類似実装・パターンを検索）
   - **ライブラリドキュメントの確認**: 使用しているフレームワークの公式ドキュメントで API 動作を確認
   - **パッケージリポジトリの検索**（Maven/npm/PyPI 等）: 新しいユーティリティを書く前に既存ライブラリを確認。実績のあるライブラリを自前実装より優先
   - **参考実装の検索**: 問題の80%以上を解決するオープンソースプロジェクトを探す
   - 要件を満たすなら、ゼロから書くより実証済みのアプローチを採用・移植する

1. **計画を立てる**
   - **planner** エージェントで実装計画を作成（3ステップ以上 or アーキテクチャに関わるタスク）
   - **architect** エージェントでシステム設計を検討（モジュール間の依存・レイヤー構成の判断時）
   - **code-architect** エージェントで具体的なクラス設計・API 設計を作成（新機能の場合）
   - 依存関係とリスクを特定
   - フェーズに分解

2. **TDD アプローチ**
   - **tdd-guide** エージェントを使用
   - テストを先に書く（RED）
   - テストを通す実装（GREEN）
   - リファクタリング（IMPROVE）
   - カバレッジ80%以上を確認
   - ビルド失敗時: **build-error-resolver** → 必要に応じて言語固有のビルドエラー解決エージェントにエスカレーション（詳細は `rules/java/agents.md` 等）
   - 反復的な修正-検証サイクルが必要な場合: **loop-operator** エージェントを使用

3. **コードレビュー**（並列実行を推奨）
   - コード記述直後に **reviewer** + **code-reviewer** エージェントを並列使用
   - フレームワーク固有の変更: フレームワーク固有のレビューエージェントを追加（詳細は言語別ルール、例: `rules/java/agents.md` の `java-reviewer`）
   - セキュリティ関連のコード: **security-reviewer** エージェントを追加
   - コメント・ドキュメンテーションコメントの変更: **comment-analyzer** エージェントを追加
   - CRITICAL・HIGH の問題を修正
   - 可能なら MEDIUM の問題も修正

4. **品質向上**（レビュー後・コミット前）
   - **code-simplifier** エージェントでコードのシンプル化を検討
   - **silent-failure-hunter** エージェントでエラーハンドリングの網羅性を検証
   - **refactor-cleaner** エージェントでデッドコード・未使用 import を整理
   - パフォーマンス懸念がある場合: **performance-optimizer** エージェントを使用

5. **敵対的レビュー**（品質向上後・コミット前。**全変更が対象。小規模修正でも省略しない**）
   - **小規模変更**: `adversarial-reviewer` 1 エージェント
   - **中規模変更**: `adversarial-reviewer` + `silent-failure-hunter` の並列実行
   - **大規模変更**: 領域別に `adversarial-reviewer` を複数並列 ＋ `meta-reviewer` でメタレビュー
   - 詳細は [adversarial-review.md](./adversarial-review.md) を参照
   - 指摘があれば実装に戻る。修正後に再度敵対的レビューを実施する

6. **ドキュメント更新**（コードの変更に応じて。対象に変更がなければスキップ）
   - 仕様書・README・CLAUDE.md・コード内コメントを同一コミットで更新する
   - 詳細は [comment-integrity.md](./comment-integrity.md) を参照

7. **ドキュメント客観レビュー**（ドキュメントを更新した場合のみ）
   - `doc-objectivity-reviewer` エージェントを使用
   - 詳細は [doc-objectivity-review.md](./doc-objectivity-review.md) を参照
   - 指摘があればドキュメントに戻る

8. **コミット & プッシュ**
   - 詳細なコミットメッセージ
   - Conventional Commits 形式に従う
   - 詳細は [git-workflow.md](./git-workflow.md) を参照

9. **マージ前チェック**
   - 全自動チェック（CI/CD）が通過していること
   - **e2e-runner** エージェントで重要なユーザーフローを検証（Web 画面に影響する変更の場合）
   - マージコンフリクトが解決済みであること
   - ターゲットブランチと同期していること
   - **doc-updater** エージェントで関連ドキュメントを更新（API・設定変更を含む場合）
