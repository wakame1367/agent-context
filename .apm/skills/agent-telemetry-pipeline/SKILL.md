---
name: agent-telemetry-pipeline
description: 複数種類のコーディングエージェント（Claude Code・Devin・Codex等）の実行ログ・スキル呼出・ツール使用を横断収集するテレメトリ基盤を作るときに使う。「エージェントのログを集約したい」「スキルが呼ばれているか計測したい」「OTelでエージェントテレメトリをまとめたい」と言われたら使う。設計は references/design.md。
---

# エージェント横断テレメトリ基盤

目的: スキル健全性（呼出率・陳腐化）や人間ボトルネック計測のため、
エージェントごとに異なるログ機構を1つのイベントストリームに正規化する。

**共通IFは「OTLP」ではなく「イベントスキーマ」。** 輸送路・バックエンドは差し替え可能な部品。

```
Claude Code ──native OTLP──┐
Codex/Cursor ─transcript tail─┼──→ otelcol → 統一スキーマ → backend
Devin Cloud ──session API poll──┤
MCP対応エージェント ──MCP report──┘
```

## 使いどきろ

- エージェントの実行テレメトリ基盤を新設・改修するとき
- スキル呼出計測・陳腐化検知・トレース採点の基盤を作るとき
- ハーネスごとのログ形式差を吸収する層を設計するとき

## 統一イベントスキーマ（資産）

```json
{
  "agent": "claude-code|devin|codex|cursor",
  "session_id": "...",
  "event": "skill_invoke|tool_use|prompt|task_end|escalation",
  "tool_name": "github-agentic-gates",
  "ts": "2026-11-23T14:00:00Z"
}
```

## ソース別の取り込み方

- **Claude Code**: ネイティブ OTLP エクスポート（`CLAUDE_CODE_ENABLE_TELEMETRY=1` + `OTEL_EXPORTER_*`）。スキル呼出は tool_use イベントのツール名
- **transcript 系**: otelcol filelog receiver で `~/.claude/projects/**/*.jsonl` 等を tail → transform で正規化
- **Devin Cloud**: session API をポーリングして OTLP HTTP でコレクタに POST
- **MCP話せるエージェント**: ローカル MCP endpoint をレポート窓口にする（プロンプトで報告させる方式。弱いが口はどのエージェントにもある）

## 設計原則

- メトリクス・ツール名属性主体で送る。プロンプト本文・コード断片は送らない（センシティブ）
- outcome 相関（修正指示・PR revert）は後付け join。v1 は invocation + session + tool_name のみ
- 時間依存記述には日付スタンプを付ける（陳腐化検出の対象にするため）

## 詳細

otelcol 設定例・poller 仕様・バックエンド選択肢・段階導入ロードマップ:
`references/design.md`
