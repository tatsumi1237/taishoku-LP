---
name: optimize-structure
description: プロジェクトの実態（言語/フレームワーク、規模、既存の.claude構成）を調査した上で、CLAUDE.md・settings.json・rules/・skills/・agents/・commands/・.mcp.json などをClaude Code公式のベストプラクティスに沿って新規作成・再編成する。「.claudeフォルダを整理して」「Claude Codeのフォルダ構成を最適化して」「プロジェクト用にClaude Codeをセットアップして」等のリクエストで使う。
disable-model-invocation: true
argument-hint: [対象ディレクトリ(省略可、既定はリポジトリルート)]
---

# フォルダ構成最適化

Claude Codeが読み込む `.claude/` 一式を、**このプロジェクトの実態に合わせて**最適化する。
固定テンプレートを機械的に流し込むのではなく、毎回「調査 → 診断 → 提案 → 実行 → 検証」の順でゼロから判断する。これにより、どんな言語・規模・セッションのプロジェクトでも対応できる。

## 原則

- 唯一の正解構成は無い。判断に迷ったら記憶で決め打ちせず、必ず [reference.md](reference.md) を読み直して公式仕様と照合する。
- シンプルさ優先。使われない空ディレクトリ、中身の薄いテンプレファイルは作らない。小規模プロジェクトなら CLAUDE.md 一枚で十分なことも多い。
- 既存の記述内容は壊さない。移動・分割する場合も情報を失わず転記する。
- 変更は必ず git で追える状態にする（明示的に頼まれない限りコミットはしない）。

## 手順

### 1. 調査

- まず [reference.md](reference.md) を読み、公式の理想構成と判断基準を確認する。
- 対象ディレクトリ（`$ARGUMENTS` があればそこ、無ければリポジトリルート）について調べる:
  - **言語/スタック**: `package.json` / `pyproject.toml` / `requirements.txt` / `go.mod` / `Cargo.toml` / `Gemfile` / `composer.json` / `*.csproj` の有無。どれも無ければ静的サイト等の非ビルド系と判断
  - **モノレポか**: `workspaces`、`packages/*`、`apps/*` 等の有無
  - **実際に使うコマンド**: `package.json` の `scripts`、`Makefile`、`.github/workflows/*` からビルド/テスト/リント/フォーマッタの実コマンドを拾う
  - **既存 `.claude/`**: 中身をGlobで一覧化。`~/.claude/` はグローバル設定なので参照のみに留め、ユーザーから明示に頼まれない限り変更しない
  - **`CLAUDE.md`**: 有無と行数（`wc -l`）
  - **`.gitignore`**: `settings.local.json` 等の除外行があるか
  - **`.mcp.json`**: 有無。チームで使う外部連携（DB/API等）の有無
  - 秘密情報が絡む設定（`.env`、資格情報）が誤って committed 対象に混ざっていないか

### 2. 診断

[reference.md](reference.md) の一覧・判断基準と現状を突き合わせ、「現状 / ギャップ / 対応要否」の表を作る。特に確認する観点:

- CLAUDE.md が無い → 作る価値があるか判断する（頻用コマンドや守るべき規約が複数あれば作成、単純な静的ページ1枚構成なら数行で十分、あるいは不要と判断してもよい）
- CLAUDE.md が200行超、または全セッションで不要な作業限定の情報が混ざっている → トピック別 `rules/*.md` への分割候補を洗い出す（`paths:` で絞れるものは絞る）
- READMEやチャットで毎回貼り付けているような複数ステップの手順（デプロイ、リリース、チェックリスト等） → `skills/` 化の候補
- 決まったBashコマンドを毎回許可していそう/確認が煩雑になりそう → `settings.json` の `permissions.allow` 候補
- フォーマッタ(prettier/black/gofmt等)がある → 編集後に自動整形する `PostToolUse` フック候補
- 個人的な認証情報やローカル限定設定が `settings.json`（committed想定）に混ざっている → `settings.local.json` へ退避し `.gitignore` を確認
- チーム共有すべき外部ツール連携があり未設定 → `.mcp.json` 候補（値は `${ENV_VAR}` 参照、秘密情報は書かない）
- 逆に、この規模のプロジェクトには過剰な構成（使われない `agents/`・`rules/` の雛形等）が無いか

### 3. 提案

診断結果を「現状 → 提案 → 理由」の短い表としてユーザーに提示してから実行する。

- 安全な追加（新規ファイル作成、無かった `.gitignore` 行の追記など）はそのまま進めてよい。
- CLAUDE.mdの大幅な書き換え、複数ファイルへの分割、既存ファイルの移動・削除など不可逆性の高い変更は、実行前にユーザーへ確認する（`AskUserQuestion` を使ってよい）。

### 4. 実行

[reference.md](reference.md) の一覧に従い、判断基準を守りながら必要なものだけを作る。

- **CLAUDE.md**: 200行目安。スタック・頻用コマンド・守るべき規約を簡潔に列挙。長くなる部分/特定作業限定の情報は `rules/` へ。
- **`.claude/rules/*.md`**: 該当ファイルを触る時だけ読ませたいものには `paths:` frontmatterを付ける。
- **`.claude/settings.json`**: 調査で確認した実コマンドに基づき `permissions.allow`（例: `Bash(npm test *)`）や `hooks`（PostToolUseでの自動整形等）を過不足なく設定。秘密情報は書かない。
- **`.claude/settings.local.json` を使う/使われうる場合**: `.gitignore` に `**/.claude/settings.local.json` が無ければ追記する。
- **`skills/` / `agents/` / `commands/`**: 実際に繰り返されている作業が確認できたものだけ作成する。空の骨組みだけのディレクトリは作らない。新規は `commands/` でなく `skills/` を使う。
- **`.mcp.json`**: チーム共有すべき外部接続がある場合のみ作成し、値は環境変数参照にする。個人用途は案内に留め、`~/.claude.json` 側を勧める（このスキルからは作成しない）。
- **`agent-memory/` 等の autogen ディレクトリ**: 雛形を手動で作らない。

### 5. 検証・報告

- `git status` / `git diff` で実際の差分を確認する。
- 何を作った/変更したか、そして何をあえて作らなかったか（理由付き）を簡潔に要約してユーザーに報告する。
