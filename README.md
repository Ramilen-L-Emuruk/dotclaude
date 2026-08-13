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
    common/                # 言語・フレームワークに依存しない汎用ルール
    java/                  # Java/Spring Boot向けの補足ルール（commonの内容を前提に差分のみ記載）
    mcp/                   # MCPツール（Context7・Playwright・Serena）の活用ガイド
  agents/
    *.md                   # 汎用的なサブエージェント定義
    java/                  # Java/Spring Boot固有のサブエージェント定義（JavaDoc・Maven・Mockito前提）
  commands/
    *.md                   # カスタムスラッシュコマンド定義
  settings.json            # permissions.allow のテンプレート（マージ用スニペット）
  skills/
    */                     # スキル定義一式（大半は外部スキル集からの複製。下記「skills/ について」参照）
```

### `rules/common/` と `rules/java/` の関係

`rules/java/` 配下のファイルは、同名の `rules/common/` ファイルを前提とした**差分**として書かれている。例えば `rules/java/testing.md` は `rules/common/testing.md` の汎用的な TDD ワークフローやエッジケースの考え方を前提に、JUnit 5 / Mockito の具体的な書き方だけを補足する構成になっている。Java 以外の言語で使う場合は `rules/common/` のみを使い、`rules/java/` は無視すればよい。他言語向けの補足ルール（`rules/python/` 等）を増やす場合も同じ構成パターンに従う。

### `agents/` と `agents/java/` の関係

`agents/` 配下は言語・フレームワークを問わず使えるように framing を汎用化したエージェント定義。`agents/java/` 配下の3ファイル（`reviewer.md`, `java-reviewer.md`, `java-build-resolver.md`）は JavaDoc・Maven・Mockito 前提の内容が本質的に Java/Spring Boot 固有のため、汎用化せずそのまま Java 向けとして残している。Java プロジェクトでは `agents/` の該当ファイルと `agents/java/` の3ファイルを両方コピーする。

### `skills/` について

`skills/` 配下30個のうち、大半（`gateguard`, `springboot-patterns`, `security-review` 等）は自作ではなく**外部のスキル集から複製したもの**。再入手可能な内容だが、環境を移す際に毎回同じスキル集を探し直す手間を省くためバックアップとして含めている。以下の3個だけは事情が異なるので個別に注意する:

- **`review`**: Java専門のコードレビューチェックリスト。自作で、`agents/java/reviewer.md` と観点が重複している
- **`continuous-learning-v2`**: セッションを観測して再利用可能なパターンを自動抽出する自律学習システム。`hooks/observe.sh` を `PreToolUse`/`PostToolUse` フックとして `settings.json` に登録しないと動作しない（スキルフォルダをコピーするだけでは有効化されない）
- **`learned/`**: `/learn` コマンドの出力先（空フォルダ）。中身は都度生成されるものなので空のままでよい

## 使い方

### 1. CLAUDE.md をコピーする場合

`CLAUDE.md` をプロジェクトルートにコピーし、末尾の「プロジェクト固有の設定」セクションに技術スタック・補助コマンド・構成メモを書き込む。それ以外のセクション（変更時の基本フロー・検証・敵対的レビュー・ドキュメント更新等）はそのまま使える想定だが、プロジェクトの運用と食い違う箇所があれば調整する。

### 2. rules をコピーする場合

必要なファイルだけを対象プロジェクトの `.claude/rules/common/`（または `rules/java/`, `rules/mcp/`）にコピーする。ファイル同士が相対リンクで参照し合っている点に注意する:

- `rules/common/` 内は同階層への参照（例: `[worktree.md](./worktree.md)`）
- `rules/java/` の各ファイルは対応する `rules/common/` ファイルへの上位階層参照（例: `[testing.md](../common/testing.md)`）を前提にした**差分**として書かれているため、`rules/java/` だけをコピーする場合は対応する `rules/common/` ファイルも必ず一緒にコピーする

コピー後は、リンク先が対象プロジェクトの `.claude/rules/` 配下に実在するか確認する。

### 3. agents をコピーする場合

使いたいエージェントだけを対象プロジェクトの `.claude/agents/` にコピーする。`.claude/rules/common/agents.md`（エージェントオーケストレーションルール）と組み合わせて使うと、どのタイミングでどのエージェントを呼ぶかの指針が揃う。

### 4. commands をコピーする場合

使いたいコマンドだけを対象プロジェクトの `.claude/commands/`（プロジェクト単位で使う場合）または `~/.claude/commands/`（全プロジェクト共通で使う場合）にコピーする。コピーすると `/learn` のようにスラッシュコマンドとして呼び出せるようになる。

### 5. settings.json の permissions.allow を使う場合

`settings.json` は**そのまま上書きコピーするファイルではなく、マージ用のスニペット**。よく使う読み取り系コマンド・Git操作・MCPツール（Playwright・Context7・Serena等）を確認プロンプト無しで許可するパターン集。

1. 対象の `~/.claude/settings.json`（全プロジェクト共通）または `.claude/settings.json` / `.claude/settings.local.json`（プロジェクト単位）を開く
2. 既存の `permissions.allow` 配列に、必要なパターンだけ追記する（既存の許可設定を上書きしないこと）
3. 破壊的なコマンド（`git push --force` 等）は意図的に含めていない。許可したい場合は個別に検討して追加する

### 6. skills をコピーする場合

使いたいスキルだけを対象プロジェクトの `.claude/skills/`（プロジェクト単位）または `~/.claude/skills/`（全プロジェクト共通）にコピーする。`continuous-learning-v2` を有効化する場合は、対応する `hooks` 設定（`settings.json` の該当セクション）も一緒に用意する必要がある。

## 注意事項

- 各ファイルはコピー元プロジェクト（Java/Spring MVC ベースのレガシー端末エミュレータ）から汎用化・言語固有部分の切り出しを行ったもの。汎用化の過程で見落としがあれば、使用時に気づいた範囲で直接修正してよい
- 社内固有の情報（ホスト名・チケット番号・個人のメモリキー参照等）は含まれない想定。コピー先プロジェクトの固有情報は各プロジェクト側で追記すること
