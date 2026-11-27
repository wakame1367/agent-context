# agent-context

エージェント横断で使うスキルの集約パッケージ
（[APM — Agent Package Manager](https://github.com/microsoft/apm) 形式）。

構成は2層:

- `.apm/skills/` — **自前スキル**（このリポジトリが本体・正）
- `apm.yml` の `dependencies.apm` — **upstream スキルへの参照**（宣言型）

このリポジトリ1つを install すれば、収録スキルがすべて consumer の
`.agents/skills/` にデプロイされ、Devin・Claude Code・Codex・Cursor・
GitHub Copilot・Gemini CLI 等どのエージェントからも読まれる。

## 使い方（consumer 側）

```bash
# apm が未導入なら: uv tool install apm-cli  (要 Python 3.11+、uv が自動調達)
cd <プロジェクトのリポジトリ>
apm install wakame1367/agent-context --target agent-skills
```

`.agents/skills/` へ各スキルが展開され、`apm.lock.yaml` が commit+hash で
ピン留めされる。次の人が `git clone && apm install` すると同一状態が再現される。

初回 install は upstream（特に `github/awesome-copilot`）の clone が重い。
以降は `apm_modules/` キャッシュが効く。

## 収録内容

### 自前スキル（`.apm/skills/`）

| スキル | 内容 |
|---|---|
| `agent-code-quality-guardrails` | 指示文より決定的な仕組み（構造・lint・検証）でエージェントのコード品質を作る方針。Lauren Tan 講演の知見 |
| `ai-coding-workflow` | エージェント前提の開発規律: タスク分解・コンテキスト管理・TDD・検証中心・環境の最小権限 |
| `spec-driven-dev` | spec を資産として扱う作法: 条項 ID 化・検証可能な仕様・spec バグの検出 |
| `skill-authoring` | SKILL.md の書き方: アトム化・有無2アーム eval・冗長/有害項目の除去 |
| `agent-fleet-ops` | 複数エージェント運用: manager/builder・レビューキャップ・トレース採点・セッション隔離 |
| `github-agentic-gates` | GitHubをエージェント駆動向けに保護・ゲート化: ruleset必須チェック・環境承認・層別レビュー・ラベル→auto-merge・非自明なハマりどころ |

末尾4つは AI Engineer チャンネル（@aiDotEngineer）の登壇 ~25件を
文字起こしして抽出した実践集。各項目の出典は
`.apm/skills/ai-coding-workflow/references/sources.md` を参照。

### upstream 参照（`dependencies.apm`）

| 領域 | スキル | 出典 |
|---|---|---|
| Actions セキュリティ | `github-actions-hardening` | github/awesome-copilot |
| Actions 効率 | `github-actions-efficiency` | 同上 |
| Actions pin 留め | `gh-actions-lock` | github/gh-actions-lock（公式 gh 拡張） |
| Dependabot | `dependabot` | github/awesome-copilot |
| ブランチ命名 | `conventional-branch` | 同上 |
| ブランチ作成 | `git-flow-branch-creator` | 同上 |
| コミット | `git-commit` | 同上 |
| レビュー対応 | `pr-address-feedback` | 同上 |
| マージ前 rebase | `rebase-current-branch` | 同上 |
| PR 作成 | `write-pr-description` | sapegin/dotfiles |
| PR の CI 修復 | `fix-pr-ci` | 同上 |
| gh CLI 規律 | `github` | 同上 |
| リリース | `github-release` | github/awesome-copilot |
| 日本語推敲 | `yomiyasu` | nanaism/yomiyasu（MIT、v3.4.1 に pin） |

`sapegin/dotfiles` のスキルはライセンス表記がないため**参照利用のみ**
（このリポジトリにコードはコピーしない）。

## スキルの追加・削除

- **自前スキル**: `.apm/skills/<name>/SKILL.md` を作って push（`includes: auto` で出荷される）
- **upstream 参照**: `apm.yml` の `dependencies.apm` を編集して push。参照形式:

- `owner/repo` — リポジトリ全体をパッケージとして
- `owner/repo/path/to/skill` — リポジトリ内の1スキルだけ
- `owner/repo#ref` — タグ・ブランチ・コミットで pin（未指定は default branch で
  drift する。consumer 側の lockfile が最終的な固定を担う）

consumer 側では `apm update` で upstream の最新を取り込む。

## License

MIT。収録スキル各々のライセンスは出典リポジトリに従う。
