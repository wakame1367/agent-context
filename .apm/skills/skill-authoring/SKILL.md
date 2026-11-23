---
name: skill-authoring
description: SKILL.md / エージェント指示書を書く・改訂するときの作法。スキル項目のアトム化、有無2アーム評価、冗長・有害項目の除去。AI Engineer「Your SKILL.md Is 70% Stuff the Model Already Knows」(Intel, Nikolay Petrov) および Skills 設計（Anthropic）に基づく。SKILL.md・AGENTS.md・CLAUDE.md・rules ファイルを作成・編集するときに使う。
---

# スキル/指示書の書き方

前提の知見（Intel 評価）: **人気スキルでも内容の ~70% はモデルが既知かどちらでも解ける項目**。書けば書くほど良いのではなく、冗長・有害な項目はコンテキストを食い、性能を下げる。

## 執筆ルール

- **モデルが知らないことだけ書く**: 一般的なベストプラクティス・言語仕様・「丁寧に書け」系は省略。リポジトリ固有の規約・非自明な手順・ツールの癖・決定済みの意思決定が価値
- **項目をアトム化する**: 1項目=1検証可能な主張。「XのときはYする」の形に分解し、箇条書きで。段落の説教ではなく手続き
- **段階的開示（progressive disclosure）**: SKILL.md は要点+トリガ条件のみ。長い参照資料・例・データは `references/` に分離してリンクし、必要時だけ読む（Anthropic Skills の設計）
- **強調の乱用を避ける**: 「IMPORTANT!!」「never do X」の連発は新しいモデルでは注意を壊し脱線の元になる。禁止は理由ごと簡潔に
- **コードから読み取れることを書かない**: ディレクトリ構造・使ってるライブラリ・命名規約は読めば分かる。読み取れない情報（なぜこの構成か、外部環境、デプロイ方法）だけ書く

## 評価（作ったあと必ずやる）

スキルはソフトウェアと同じく eval で品質を測る:

1. 各アトム項目に「これを知らないと答えられない/実行できないタスク」を1つ作る
2. そのタスクを **スキル有り/無しの2アーム** で実行し、出力を LLM ジャッジ（できれば別系統モデル）で採点
3. 4象限で分類:
   - 有無どちらでも解ける → **削る**（無駄。典型70%）
   - **有ると性能が下がる** → 即刻修正・削除
   - 無だと解けず有だと解ける → 本来の価値。残す
   - どちらでも解けない → 書き方が悪い or タスク設計が悪い。書き直して再評価
4. コストは小さい（~30スキルの評価で $14 規模）。新規追加・モデル更新時に再評価する

## 運用

- モデル更新・ツール更新のたびに棚卸しする（前年のベストプラクティスが障害になる例が多発した）
- スキル間で重複する内容は片方に寄せる
- エージェントが誤用した事例があれば、文言修正より先に「構造・lint・ツールで防げないか」を検討する

## 出典

- Nikolay Petrov (Intel) — "Your SKILL.md Is 70% Stuff the Model Already Knows"（アトム化・2アームeval・4象限）
- Barry Zhang & Mahesh Murag (Anthropic) — "Don't Build Agents, Build Skills Instead"（フォルダ構造・段階的開示・skill-creator）
- Lydia Hallie (Anthropic) — "Rethinking Claude Code Best Practices"（CLAUDE.md 衛生・`doctor prompt audit`）
- 詳細: `~/work/ai-engineer/`（全動画カタログ・トランスクリプト）
