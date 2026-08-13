# ワークツリー運用

## 作成手順

`EnterWorktree(name: ...)` はブランチ名を自動変換してしまうため使用しない。
必ず以下の 2 ステップで行う。

```bash
# Step 1: 正しいブランチ名でワークツリーを作成
git worktree add -b worktree/<type>/<name> .claude/worktrees/<name>

# Step 2: 作成したワークツリーに入る（path 指定で呼ぶ）
EnterWorktree(path: ".claude/worktrees/<name>")
```

type の種類: `feat` / `fix` / `refactor` / `docs` / `chore`

## 削除手順

ワークツリー内にいる場合は先に抜けてから削除する。

```bash
# Step 1: ワークツリーから抜ける（ディレクトリは保持）
ExitWorktree(action: "keep")

# Step 2: ワークツリーディレクトリを削除
git worktree remove .claude/worktrees/<name>

# Step 3: ブランチを削除
git branch -d worktree/<type>/<name>
```
