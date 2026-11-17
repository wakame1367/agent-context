# agent-context

エージェント横断で使う GitHub 開発ワークフロー系スキルの**宣言型メタパッケージ**
（[APM — Agent Package Manager](https://github.com/microsoft/apm) 形式）。

スキル本体は upstream リポジトリを参照し、`apm.yml` の `dependencies.apm` が
正（source of truth）。このリポジトリ1つを install すれば、収録スキルがすべて
consumer の `.agents/skills/` にデプロイされ、Devin・Claude Code・Codex・
Cursor・GitHub Copilot・Gemini CLI 等どのエージェントからも読まれる。

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

`sapegin/dotfiles` のスキルはライセンス表記がないため**参照利用のみ**
（このリポジトリにコードはコピーしない）。

## スキルの追加・削除

`apm.yml` の `dependencies.apm` を編集して push。参照形式:

- `owner/repo` — リポジトリ全体をパッケージとして
- `owner/repo/path/to/skill` — リポジトリ内の1スキルだけ
- `owner/repo#ref` — タグ・ブランチ・コミットで pin（未指定は default branch で
  drift する。consumer 側の lockfile が最終的な固定を担う）

consumer 側では `apm update` で upstream の最新を取り込む。

## License

MIT。収録スキル各々のライセンスは出典リポジトリに従う。
