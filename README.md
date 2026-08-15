# dotclaude

複数マシン・複数プロジェクトで再利用する Claude Code 設定一式（`CLAUDE.md` テンプレート・ルール・エージェント定義・カスタムコマンド）を集約したリポジトリ。

**想定読者**: 複数マシン・複数プロジェクトで作業する自分自身（将来の自分）。他の人と共有する場合も、以下の運用方針・使い方を前提に読んでもらえば迷わないはず。

## 運用方針

このリポジトリは **clone してから必要な内容だけをつまみ食いする** ことを想定している。symlink 等による自動同期は行わない。

1. このリポジトリを clone する
2. 使いたいファイルだけを対象プロジェクトの `.claude/` 配下・プロジェクトルートにコピーする
3. コピーしたファイルをプロジェクトの実情に合わせて調整する（プレースホルダーのコマンド例・技術スタックの記述などを実際の内容に書き換える）

コピー後にこのリポジトリと同期を取り続ける仕組みは無いため、良い変更を思いついたらこのリポジトリ側にも書き戻しておくと、次にコピーするときに活かせる。

## ディレクトリ構成

```
dotclaude/
  README.md              # このファイル
  CLAUDE.md               # プロジェクトCLAUDE.mdのテンプレート（プレースホルダー入り）
  rules/
    common/                # 言語・フレームワークに依存しない汎用ルール（常時読み込み）
    java/                  # Java/Spring Boot向けの補足ルール（commonの内容を前提に差分のみ記載）
    dotnet/                # C#/.NET向けの補足ルール（同上）
    mcp/                   # MCPツール（Context7・Playwright・Serena）の活用ガイド
  agents/
    *.md                   # 汎用的なサブエージェント定義
    java/                  # Java/Spring Boot固有のサブエージェント定義（JavaDoc・Maven・Mockito前提）
  commands/
    *.md                   # カスタムスラッシュコマンド定義
  settings.json            # ~/.claude/settings.json のテンプレート（permissions・model・hooks 等）
  statusline-command.sh    # ステータスライン表示スクリプト（モデル名・コンテキスト使用率・Gitブランチ）
  .gitignore               # 機密・実行時データの混入を防ぐ除外パターン
  skills/
    */                     # スキル定義一式（大半は外部スキル集からの複製。下記「skills/ について」参照）
```

### `rules/common/` と言語別ルールの関係

`rules/java/` `rules/dotnet/` 配下のファイルは、同名の `rules/common/` ファイルを前提とした**差分**として書かれている。例えば `rules/java/testing.md` は `rules/common/testing.md` の汎用的な TDD ワークフローやエッジケースの考え方を前提に、JUnit 5 / Mockito の具体的な書き方だけを補足する。`rules/dotnet/testing.md` は同じ位置づけで xUnit の書き方を補足する。他言語向けの補足ルール（`rules/python/` 等）を増やす場合も同じ構成パターンに従う。

#### 言語別ルールは frontmatter で読み込みを絞る（重要）

言語別ルールの各ファイル先頭には、対象ファイルを限定する YAML frontmatter が入っている:

```yaml
---
paths:
  - "**/*.cs"
  - "**/*.csproj"
---
```

これにより、C# を触っているセッションでは `rules/dotnet/` だけが読み込まれ、`rules/java/` の Maven や JPA の話は流れ込まない。**frontmatter を外すと全プロジェクトで常時読み込まれ、無関係な言語のルールがコンテキストを圧迫する**ので、コピー時・追記時に消さないこと。`rules/common/` と `rules/mcp/` は逆に、言語を問わず常時読み込ませたいので frontmatter を付けない。

### `agents/` と `agents/java/` の関係

`agents/` 配下は言語・フレームワークを問わず使えるように framing を汎用化したエージェント定義。`agents/java/` 配下の3ファイル（`reviewer.md`, `java-reviewer.md`, `java-build-resolver.md`）は JavaDoc・Maven・Mockito 前提の内容が本質的に Java/Spring Boot 固有のため、汎用化せずそのまま Java 向けとして残している。Java プロジェクトでは `agents/` の該当ファイルと `agents/java/` の3ファイルを両方コピーする。

### `skills/` について

`skills/` 配下の大半（`gateguard`, `springboot-patterns`, `security-review` 等）は自作ではなく**外部のスキル集から複製したもの**。再入手可能な内容だが、環境を移す際に毎回同じスキル集を探し直す手間を省くためバックアップとして含めている。以下の4個だけは事情が異なるので個別に注意する:

- **`release`**: 自作。**単体では機能しない**（完了ブランチを `ready/*` へ改名する規約が前提のため）。導入経路によって必要な組み合わせが変わる:
  - `CLAUDE.md` テンプレートを使う場合 — `CLAUDE.md`（「リリース」節に規約を内包）＋ このスキル
  - `rules/common/` をモジュールとして使う場合 — `worktree.md`（`ready/` 規約）＋ `development-workflow.md`（手順9〜10）＋ `git-workflow.md`（マージ・版上げの委譲先）＋ このスキル
  - 両方使う場合は上記すべて

  コピー後に置き換えるべきもの（既定ブランチ名・各種コマンド・検証手段）は SKILL.md 冒頭の「コピー時に置き換えるもの」に列挙してある
- **`review`**: Java専門のコードレビューチェックリスト。自作で、`agents/java/reviewer.md` と観点が重複している
- **`continuous-learning-v2`**: セッションを観測して再利用可能なパターンを自動抽出する自律学習システム。`hooks/observe.sh` を `PreToolUse`/`PostToolUse` フックとして `settings.json` に登録しないと動作しない（スキルフォルダをコピーするだけでは有効化されない）
- **`learned/`**: `/learn` コマンドの出力先。中身は都度生成されるものなので空のままでよい。git は空ディレクトリを追跡しないため `.gitkeep` を置いてフォルダだけ再現されるようにしている

## 使い方

### 1. CLAUDE.md をコピーする場合

`CLAUDE.md` をプロジェクトルートにコピーし、末尾の「プロジェクト固有の設定」セクションに技術スタック・補助コマンド・構成メモを書き込む。それ以外のセクション（変更時の基本フロー・検証・敵対的レビュー・ドキュメント更新等）はそのまま使える想定だが、プロジェクトの運用と食い違う箇所があれば調整する。

### 2. rules をコピーする場合

必要なファイルだけを対象プロジェクトの `.claude/rules/common/`（または `rules/java/`, `rules/dotnet/`, `rules/mcp/`）にコピーする。ファイル同士が相対リンクで参照し合っている点に注意する:

- `rules/common/` 内は同階層への参照（例: `[worktree.md](./worktree.md)`）
- `rules/java/` `rules/dotnet/` の各ファイルは対応する `rules/common/` ファイルへの上位階層参照（例: `[testing.md](../common/testing.md)`）を前提にした**差分**として書かれているため、言語別ルールだけをコピーする場合は対応する `rules/common/` ファイルも必ず一緒にコピーする

コピー後は次の2点を確認する:

1. リンク先が対象プロジェクトの `.claude/rules/` 配下に実在すること
2. 言語別ルールの frontmatter（`paths:`）が残っていること。消えていると無関係な言語のルールが常時読み込まれる（上記「言語別ルールは frontmatter で読み込みを絞る」参照）

### 3. agents をコピーする場合

使いたいエージェントだけを対象プロジェクトの `.claude/agents/` にコピーする。`.claude/rules/common/agents.md`（エージェントオーケストレーションルール）と組み合わせて使うと、どのタイミングでどのエージェントを呼ぶかの指針が揃う。

### 4. commands をコピーする場合

使いたいコマンドだけを対象プロジェクトの `.claude/commands/`（プロジェクト単位で使う場合）または `~/.claude/commands/`（全プロジェクト共通で使う場合）にコピーする。コピーすると `/learn` のようにスラッシュコマンドとして呼び出せるようになる。

### 5. settings.json を使う場合

`~/.claude/settings.json`（全プロジェクト共通の設定）のテンプレート。**そのまま上書きコピーせず、必要な項目だけを既存の設定にマージする**（上書きすると既存の許可設定や個人設定が消える）。

収録内容:

| 項目 | 内容 |
|------|------|
| `permissions.allow` | よく使う読み取り系コマンド・Git 操作・MCP ツール（Playwright・Context7・Serena 等）を確認プロンプト無しで許可するパターン集 |
| `permissions.ask` | `git push --force` 系は明示的に確認を求める（履歴を壊しうるため、`allow` には入れていない） |
| `model` / `language` / `effortLevel` / `tui` 等 | 個人の好みの設定。**そのまま使う前に自分の好みに合わせて調整すること** |
| `statusLine` | `statusline-command.sh` の登録。下記「7. statusline-command.sh を使う場合」を先に実施しないと動作しない |
| `hooks` | `skills/continuous-learning-v2/hooks/observe.sh` の登録。**このスキルを `~/.claude/skills/` にコピーしていないとフックが毎回失敗する** |

`statusLine` と `hooks` のコマンドパスは `~/.claude/...` 起点で書いてある。別の場所に置く場合は書き換えること。

### 6. skills をコピーする場合

使いたいスキルだけを対象プロジェクトの `.claude/skills/`（プロジェクト単位）または `~/.claude/skills/`（全プロジェクト共通）にコピーする。`continuous-learning-v2` を有効化する場合は、対応する `hooks` 設定（`settings.json` の該当セクション）も一緒に用意する必要がある。

### 7. statusline-command.sh を使う場合

ステータスラインにモデル名・思考の深さ（effort）・コンテキスト使用率（残量に応じて緑／黄／赤）・Git ブランチ名（未コミット変更があれば `*`）を表示するスクリプト。

1. `~/.claude/statusline-command.sh` にコピーする
2. 実行権限を付ける: `chmod +x ~/.claude/statusline-command.sh`
3. `~/.claude/settings.json` に登録する:
   ```json
   "statusLine": { "type": "command", "command": "~/.claude/statusline-command.sh" }
   ```

`bash` と `python3`（JSON パースに使用）が PATH 上にあることが前提。

## 注意事項

- 各ファイルはコピー元プロジェクト（Java/Spring MVC ベースのレガシー端末エミュレータ）から汎用化・言語固有部分の切り出しを行ったもの。汎用化の過程で見落としがあれば、使用時に気づいた範囲で直接修正してよい
- 社内固有の情報（ホスト名・チケット番号・個人のメモリキー参照等）は含まれない想定。コピー先プロジェクトの固有情報は各プロジェクト側で追記すること
- **このリポジトリは公開されている**。認証情報（`.credentials.json`）・会話履歴（`history.jsonl`）・組織のポリシー設定（`remote-settings.json` 等）・セッションデータは絶対にコミットしない。`.gitignore` に歯止めを入れてあるが、新しくファイルを追加するときは中身を必ず確認すること
- 個人のペルソナ設定（`~/.claude/CLAUDE.md` のキャラクター設定・`personas/`・`hooks/random-persona.sh`）は意図的に含めていない。必要なら各自の環境で用意すること
