# Git ワークフロー

## コミットメッセージ形式
```
<type>: <description>

<optional body>

Co-Authored-By: Claude <モデル名> <noreply@anthropic.com>
```

タイプ: feat, fix, refactor, docs, test, chore, perf, ci

コミットメッセージ末尾には必ず `Co-Authored-By` を付与する。**モデル名はその時作業している Claude モデルに合わせる**（ハーネスが指定するモデル名を使用する。特定モデルに固定しない）。
- 例（Sonnet 4.6 で作業時）: `Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>`

## コミット前の確認

- **コミット前に必ずユーザーに確認を取る。省略しない**
- 何を・なぜ・どう直したかを簡潔に提示し、コミットしてよいか確認する
- 同一セッション内で別の修正のコミット許可をもらっていても、その修正を個別に確認していない場合は改めて確認する

## プルリクエストワークフロー

PR 作成時:
1. 最新コミットだけでなく、全コミット履歴を分析
2. `git diff [base-branch]...HEAD` で全変更を確認
3. 包括的な PR サマリーを作成
4. テスト計画を TODO として含める
5. 新しいブランチの場合は `-u` フラグでプッシュ

> 計画・TDD・コードレビューを含む完全な開発プロセスは
> [development-workflow.md](./development-workflow.md) を参照。
