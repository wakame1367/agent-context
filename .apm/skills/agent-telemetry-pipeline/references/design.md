# agent-telemetry-pipeline 設計書

複数種類のコーディングエージェントの実行テレメトリを、共通イベントスキーマに
正規化して蓄積する基盤の設計。バックエンドは **Cloudflare R2**（S3互換）を正とする。

決定事項（2026-11-23）:
- 溜め先は R2（課金ほぼゼロ・保持無制限・複数マシン横断集計可）
- マネージドOTel（Grafana/Axiom）は保持延長で実質 $19–25/月〜になるため不採用
- ローカルファイルは R2 到達不能時の冗長として併用してよい

## 目的

- スキル健全性の計測（スキル別呼出率・未使用スキル検出・呼出後の修正相関）
- エージェント活動の横断可視化（将来: 人間ボトルネック計測・トレース採点の入力）

## アーキテクチャ

```
Claude Code ──native OTLP──┐
Codex/Cursor ─transcript──┤
        filelog receiver  ├─→ otelcol (WSL・contrib版・単一バイナリ)
Devin Cloud ──poller script─┤      │  transform → 統一スキーマ
MCP対応Agent ──MCP report───┘      ├─→ awss3exporter → Cloudflare R2  (正)
                                   └─→ file exporter → ローカル (冗長・任意)
```

- 共通IF = **イベントスキーマ**。輸送（OTLP/HTTP/file）と保管（R2/local）は差し替え可能な部品
- otelcol は contrib 版（awss3exporter・filelogreceiver・transformprocessor が必要）
- 複数マシン: 各マシンの otelcol が同じ R2 エンドポイントに書く。Cloud セッション側も同様

## 統一イベントスキーマ（v1）

```json
{
  "agent": "claude-code|devin|codex|cursor|other",
  "session_id": "...",
  "event": "skill_invoke|tool_use|prompt_meta|task_end|escalation",
  "tool_name": "github-agentic-gates",
  "host": "wsl-main|windows|devin-cloud",
  "ts": "2026-11-23T14:00:00Z"
}
```

設計判断:
- `prompt` イベントはメタのみ（本文を送らない。長さ・turn番号程度）
- 属性は最小に絞る。詳細は join キー（session_id）だけ残して生トランスクリプト側に置く
- v2 で outcome 相関（ユーザーの修正指示・PR revert・task成否）を join する設計に備え、
  session_id はリポジトリ/PR/チケットIDと結びつけられる形にする

## ソース別の取り込み

### Claude Code（最も簡単）

環境変数でネイティブOTLPをコレクタへ:

```bash
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318   # ローカル otelcol
```

スキル呼出は tool イベントのツール名属性として届く想定（要実機確認。届かなければ
jsonl tail にフォールバック）。

### transcript tail（Codex 等・フォールバック共通）

otelcol の filelog receiver で JSONL を tail し、transform で正規化。
対象パス例: `~/.claude/projects/**/*.jsonl`（Claude側のフォールバック兼用）

### Devin Cloud

定期ポーリングスクリプト（cron/launchd）。Devin API からセッション一覧・状態を取得し、
統一スキーマのイベントを OTLP HTTP で localhost:4318 に POST。
※ Devin 側で取得できる粒度は実調査が必要（セッションAPIの応答項目）

### MCP report 窓口（予備）

OTLPを出せないが MCP が話せるエージェント向けに、ローカル MCP endpoint を
「レポート受付」として用意し、プロンプトで報告させる方式。
（Orca記事の Codex→worker 送信と同型。弱い経路なので主要経路にはしない）

## otelcol 設定の骨子

```yaml
receivers:
  otlp:
    protocols:
      grpc: {endpoint: 0.0.0.0:4317}
      http: {endpoint: 0.0.0.0:4318}
  filelog:
    include: [/home/wakame/.claude/projects/**/*.jsonl]
    start_at: end

processors:
  transform/normalize:
    # ソース別の写像ルール（agent/session/event/tool_name/ts への正規化）
  batch:

exporters:
  awss3/r2:
    s3uploader:
      region: auto
      endpoint: https://<account-id>.r2.cloudflarestorage.com
      bucket: agent-telemetry
      s3_force_path_style: true
    # 認証情報は環境変数 (AWS_ACCESS_KEY_ID / AWS_SECRET_ACCESS_KEY) で注入
  file/local:           # 冗長・障害時バッファ（任意）
    path: /var/lib/agent-telemetry/buffer.jsonl

service:
  pipelines:
    logs:
      receivers: [otlp, filelog]
      processors: [transform/normalize, batch]
      exporters: [awss3/r2, file/local]
```

## R2 側の構成

- バケット: `agent-telemetry`（新規作成）。パーティションは日付/エージェント単位
  （`awss3exporter` の prefix 設定で `logs/agent=.../year=.../month=.../day=.../`）
- 認証: R2 API トークン（S3互換・最小権限で当該バケットのみ書き込み可）を発行し、
  コレクタの環境変数に入れる。1Password保管・`op run` 経由で注入するのがこの環境の作法
- コスト見積: 月数十〜数百MB想定 → 実質¥0〜50/月

## クエリ・可視化（後付け）

Cold層を正にしたので「見る」は別系統で後付けする:

- 即時: R2のJSONLを DuckDB で直接読む（`s3://` 対応。集計・スキル別呼出ランキング等）
- 月次レポート: cronで「スキル別 invocation 数・前期比・ゼロ呼出一覧」を生成して
  agent-context 等のリポジトリに issue 起票
- 可視化が欲しくなったら: Grafana OSS を自前で立てて R2/DuckDB に向ける、
  または Cloudflare Workers Analytics Engine を metrics 用に併用

## センシティブ対策

- 送信するのはメトリクス相当（ツール名・イベント種別・時刻・成否フラグ）
- プロンプト本文・コード断片・ファイルパスの個人情報は送らない
- 送信前に transform で既知のセンシティブパターンをドロップ（トークンらしき文字列等）

## 段階導入ロードマップ

1. **v0（最速・検証）**: コレクタ無し。cron が `~/.claude/**/*.jsonl` を直接集計して
   スキル別呼出レポートを出す。価値の実証を先に
2. **v1**: otelcol 起動。Claude Code の OTLP + filelog を R2 に流す。Devin poller 追加
3. **v2**: outcome 相関（GitHub API で session→PR→revert を join）。
   月次スキル健全性レポートを自動起票
4. **v3**: 必要になったらダッシュボード（Grafana OSS or Cloudflare AE）

## 未決定・実調査が要る点

- Claude Code OTLP に「スキル名」が本当に含まれるか（含まれなければ filelog 側が本線）
- Devin Cloud のセッションAPIで取れるイベント粒度
- 複数マシンで otelcol を常駐化する方法（WSL側に systemd user unit or 起動時スクリプト）
- R2 のオブジェクト整理ポリシー（何年残すか・削除ルール。保持無制限でも整理は要る）
