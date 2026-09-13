# Claude Code `.claude/` 理想構成リファレンス

Claude Code公式ドキュメント（`.claude ディレクトリを探索する` / `スキルで Claude を拡張する`）準拠。
SKILL.md の「診断」「実行」で必ずこの表と照らし合わせる。記憶や思い込みで判断しない。

凡例: **committed**=リポジトリにコミットする / **gitignored**=Claude Codeが自動でgit除外する / **local**=個人環境のみで共有しない / **autogen**=Claudeが自動生成・更新するので手動で雛形を作らない

## プロジェクトルート（gitリポジトリ直下）

| パス | 分類 | 読み込まれるタイミング | 用途 |
|---|---|---|---|
| `CLAUDE.md`（`.claude/CLAUDE.md` でも可） | committed | 毎セッション開始時に必ず読み込まれる | プロジェクト規約・頻用コマンド・アーキテクチャの要点。**200行を目安**に収める。長い/特定ファイル操作時だけ要る内容は `rules/` かスキルへ逃がす |
| `.mcp.json` | committed | セッション開始時にサーバーへ接続（ツール定義自体は遅延ロード） | チーム共有のMCPサーバー設定。秘密情報は `${ENV_VAR}` 参照にし、値は書かない。個人用サーバーは `~/.claude.json`（`claude mcp add --scope user`）へ |
| `.worktreeinclude` | committed | `git worktree` 作成時 | gitignore対象だが新しいworktreeにコピーしたいファイル（`.env` 等）を `.gitignore` と同じ構文で列挙 |
| `.gitignore` | committed | — | `**/.claude/settings.local.json` 等、個人設定を除外する行を含める |

## `.claude/`（プロジェクト、原則コミット）

| パス | 分類 | 読み込まれるタイミング | 用途 |
|---|---|---|---|
| `settings.json` | committed | 毎セッション。`~/.claude/settings.json` を上書きし、`settings.local.json`・CLI引数・管理設定がさらに上書き | `permissions`(許可/拒否コマンド)、`hooks`(PostToolUse等でのフォーマッタ実行等)、`statusLine`、`model`、`env`、`outputStyle`。**Claudeの判断でなく強制されるルール**を置く場所 |
| `settings.local.json` | gitignored | 個人設定として最優先で適用 | チーム設定に追加したい自分専用の許可コマンド等。Claude Codeが書き込む際、自動で `.gitignore`（または global excludes）に追記される |
| `rules/*.md`（サブディレクトリ可） | committed | `paths:` frontmatter無し→セッション開始時に常時読込。`paths:` 有り→該当globにマッチするファイルを開いた時だけ読込 | CLAUDE.mdから分割したトピック別・パス限定の規約（例: `rules/testing.md` に `paths: ["**/*.test.ts"]`）。CLAUDE.mdが200行に近づいたら分割を検討 |
| `skills/<name>/SKILL.md`（+付随ファイル） | committed | `/skill-name` 呼び出し時、またはdescriptionにClaudeが一致すると判断した時 | 「何度もチャットに貼り付けている手順」「CLAUDE.mdの一節が手順化してきた」ものを切り出す。副作用のある操作（デプロイ等）は `disable-model-invocation: true` でユーザー専用に。テンプレ/チェックリスト/スクリプトを同ディレクトリに同梱可 |
| `commands/*.md` | committed | `/command-name` 呼び出し時 | 旧来の単一ファイルコマンド。**新規はskills/を推奨**（付随ファイルを同梱できるため）。同名ならskillsが優先 |
| `output-styles/*.md` | committed | 選択中のスタイルとして全応答に適用 | チームで共有する出力スタイル（レビューモード等）。個人用は `~/.claude/output-styles/` へ |
| `agents/*.md` | committed | Claudeが委任判断した時、または `@agent-name` で明示指定 | 独立コンテキストで動くサブエージェント定義。`tools:` でRead/Grep等に制限し安全に分離（例: コードレビュー専用） |
| `workflows/*.js` | committed | 起動時にロードされ `/<name>` になる | 複数サブエージェントを調整する動的ワークフロー。`/workflows` から保存して作るもので、ゼロから手書きしない |
| `agent-memory/<agent>/MEMORY.md` | committed・**autogen** | サブエージェント起動時にシステムプロンプトへ注入 | `memory: project` を設定したサブエージェントが自分で書き込む記憶。**手動で雛形を作らない**。チーム非共有にしたい場合はagent側で `memory: local`（`.claude/agent-memory-local/` へ、要gitignore）、プロジェクト超えて使うなら `memory: user`（`~/.claude/agent-memory/`） |

## `~/`（ユーザーのホーム、全プロジェクト共通・非コミット）

| パス | 分類 | 用途 |
|---|---|---|
| `~/.claude.json` | local | テーマ、OAuth、プロジェクトごとの信頼状態、**個人用**MCPサーバー。主に `/config` 経由で編集 |
| `~/.claude/CLAUDE.md` | local | 全プロジェクト共通の個人的な好み（応答スタイル、コミット規約等）。短く保つ。プロジェクト側の記述が競合時は優先される |
| `~/.claude/settings.json` | local | 全プロジェクトのデフォルト設定。プロジェクト側 `settings.json` が同じキーを上書き |
| `~/.claude/rules/` `~/.claude/skills/` `~/.claude/commands/` `~/.claude/agents/` `~/.claude/output-styles/` `~/.claude/workflows/` | local | プロジェクト版と同構造で、個人が全プロジェクトで使うもの |
| `~/.claude/agent-memory/` `~/.claude/projects/<proj>/memory/` | local・autogen | サブエージェント記憶（user scope）/ メインセッションのauto memory。**Claudeが自動生成・更新するので手動作成しない** |

## 判断基準（このスキルが従うルール）

1. **空の骨組みを作らない。** `skills/`, `agents/`, `rules/` は実際に繰り返し使う中身が見えてから作る。中身の無いディレクトリ構成だけ整えても価値がない。
2. **CLAUDE.mdは短く。** 目安200行。全セッションでロードされるため、特定作業でしか要らない情報は `rules/`（パス限定）かスキル（呼ばれた時だけ）に逃がす。
3. **強制したいなら settings.json（hooks/permissions）、伝えたいなら CLAUDE.md/rules。** Claudeへの「お願い」と「強制」を混同しない。
4. **個人情報とチーム情報を分離。** 秘密情報・個人専用許可は `settings.local.json` / `~/.claude/` 側。チーム共有はプロジェクト `.claude/` 側かつ秘密情報を含めない。
5. **autogenディレクトリ（`agent-memory/`, `projects/*/memory/`）は手動で雛形を作らない。** Claude自身が必要になった時に作る。
6. **小規模・単純なプロジェクトでは最小構成で十分。** 静的サイトやツールが無いリポジトリに大掛かりな `rules/` `agents/` 体系は過剰。CLAUDE.md一枚、必要なら`settings.json`だけで良いことも多い。
