---
name: github-agentic-gates
description: GitHubリポジトリをエージェント駆動開発向けに整備するときに使う。必須CIチェック・Environment承認ゲート・CODEOWNERS層別レビュー・人間ラベル起動のauto-merge・エビデンスパケットの設定手順と非自明なハマりどころを収録。「rulesetを作って」「ブランチ保護を設定して」「エージェントのPRをゲート付きで自動マージしたい」「GitHubで人間レビューを減らしたい」と言われたら使う。
---

# GitHub のエージェント駆動ゲート設計

機械評価できるチェックは必須ステータス化し、人間は「意図の承認」と「リスク受容」の
2ゲートに集中させる。プロンプトの禁止ではなく ruleset・環境保護・権限で物理的に縛る。

動作確認済みの完全な実装例: https://github.com/wakame1367/agentic-sandbox

## 使いどきろ

- 新規・既存リポジトリをエージェント開発向けに保護設定するとき
- PR の人間レビューを機械ゲート＋最小承認に置き換えたいとき
- `gh api` で ruleset・環境保護をコードから適用したいとき

## 適用する構成

### 1. 必須CIチェック（ruleset）

job名を context に持つ required_status_checks。
`strict_required_status_checks_policy: true` で最新化必須（merge queue 的な安全性の簡易版）。

```bash
gh api -X POST repos/{owner}/{repo}/rulesets --input - <<'EOF'
{"name":"protect-main","target":"branch","enforcement":"active",
 "conditions":{"ref_name":{"include":["~DEFAULT_BRANCH"],"exclude":[]}},
 "rules":[{"type":"non_fast_forward"},{"type":"deletion"},
  {"type":"required_status_checks","parameters":{
   "required_status_checks":[{"context":"lint"},{"context":"test"},{"context":"verify"}],
   "strict_required_status_checks_policy":true}}]}
EOF
```

### 2. 環境ゲート（委譲の境界）

staging は無保護で自律、production に required reviewers を設定。
デプロイジョブに `environment: production` を書けば保護ルールで自動停止し、
承認は `POST .../actions/runs/{run_id}/pending_deployments`（API・モバイルから可）。

```bash
# reviewers には login ではなく user id を渡す
gh api -X PUT "repos/{owner}/{repo}/environments/production" --input - <<EOF
{"reviewers":[{"type":"User","id":$(gh api user --jq .id)}],
 "wait_timer":0,"prevent_self_review":false}
EOF
```

単独運用では `prevent_self_review: false` 必須（true だと自分のデプロイを承認できない）。

### 3. 層別レビュー（CODEOWNERS）

人間必須領域だけを宣言（例: `/src/core/ @owner`）。
全面必須レビューは人間ボトルネックに逆戻りするのでしない。

### 4. 人間ラベル → auto-merge（ラベルゲート）

`pull_request: [labeled]` で起動する workflow。

```yaml
if: >-
  github.event.label.name == 'merge-ready'
  && github.event.sender.type == 'User'
steps:
  - run: gh pr merge --auto --squash --repo "${{ github.repository }}" "${{ github.event.pull_request.number }}"
    env:
      GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

- bot/app が付けたラベルは `sender.type == 'Bot'` で無視される
- 事前に `gh api -X PATCH repos/{o}/{r} -f allow_auto_merge=true` が必要

### 5. チケット標準形（入力側）

- issue form で「ゴール / spec条項ID / 検証手段 / エスカレーション条件 / リスク分類」を必須化
- PR テンプレは検証結果の添付欄を持つ（エビデンスパケット）
- 雛形: agentic-sandbox の `.github/ISSUE_TEMPLATE/`・`pull_request_template.md`

## ハマりどころ（実機で確認済み）

- **`secrets.GITHUB_TOKEN` でマージすると push イベントが発行されず後続ワークフローが起動しない**
  （無限ループ防止の仕様）。対処: `pull_request.closed` で拾うか GitHub App トークンでマージ
- **`pull_request` 系イベントが参照するワークフロー定義はベースブランチ側**。
  トリガー追加を含む PR のマージ自体では新トリガーは発火しない（次回から有効）
- **ワークフロー内の `gh` は checkout 無しではリポを解決できない** → `--repo` を渡す
- **environment の reviewers は user id 指定**。login 文字列は受け付けない
- **個人リポジトリで必須PRレビュー（承認数≥1）は自爆する** — 自分のPRを自分で承認できない。
  単独運用では status checks のみにして承認は環境ゲート側に寄せる

## プラン制約

- public リポジトリは Free で全機能（rulesets・environments・merge queue 含む）が使える
- private × 個人: 保護系は Pro が必要、merge queue は個人では不可（組織 Team 以上）
- CodeQL・secret scanning の private 利用は GHAS。代替は Semgrep 等を Actions で回す

## 関連

- `agent-code-quality-guardrails`: このスキルは「信頼のピラミッド」（決定的チェック > 検証 > 文章 > 人間レビュー）の GitHub 実装版
