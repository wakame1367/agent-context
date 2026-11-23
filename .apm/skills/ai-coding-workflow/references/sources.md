# 出典トーク一覧（ai-coding-workflow / skill-authoring / spec-driven-dev）

すべて AI Engineer チャンネル https://www.youtube.com/@aiDotEngineer の登壇動画。
文字起こしは `~/work/ai-engineer/transcripts/<videoID>.md.gz`（公式アーカイブ、自動書き起こしのため誤記あり）。
読むときは `zcat` / `zless`、検索は `zgrep` / `rg -z` が使える。
全動画カタログ（1,519本・分類・取得状況付き）は `~/work/ai-engineer/CATALOG.md`。
CODE 2026 の登壇別トランスクリプトは `~/work/ai-engineer/code2026/*.md`。

| 動画 | ID | 主な寄与 |
|---|---|---|
| Don't Build Agents, Build Skills Instead — Barry Zhang & Mahesh Murag (Anthropic) | CEvIs9y1uog | スキル=フォルダ、段階的開示、MCP=接続/スキル=専門性 |
| How We Build Effective Agents — Barry Zhang (Anthropic) | D7_ipDqhtwk | エージェント化の判断基準、シンプル主義、エージェント視点デバッグ |
| The New Code — Sean Grove (OpenAI) | 8rABwKRsec4 | spec=資産、コードは不可逆な射影、ID化・検証可能なspec |
| Software Fundamentals Matter More Than Ever — Matt Pocock | v4F1gFy-hqg | specs-to-code批判、深いモジュール、インターフェース設計で委譲 |
| Full Walkthrough: Workflow for AI Coding — Matt Pocock | -QFHIoCo-Ko | スマートゾーン/ダムゾーン、Grill Me、イシューDAG、TDD |
| Rethinking Claude Code Best Practices — Lydia Hallie (Anthropic) | vlfCjWuv4es | CLAUDE.md衛生、denyルール/サンドボックス、hooks検証、auto mode |
| All Code Is Slop (Except Mine) — Lauren Tan (SpaceXSI) | O_zcc6Xl12g | 環境の逆ピラミッド、lint/API設計でルール化、検証成果物レビュー |
| 40x More Merged PRs With 2.7x the Engineers — Walden Yan (Cognition) | U92J5mCDLMc | Software Factory、小pod、ジョブの与え方、使い捨てVM |
| Your SKILL.md Is 70% Stuff the Model Already Knows — Nikolay Petrov (Intel) | yzx4IuEh_tU | アトム化、2アームeval、4象限 |
| 225,000 Lines of Tests, Written and Merged by Agents — Kush Agrawal (Duolingo) | D_4Rn6uLdQM | テスト執筆をPRライフサイクル化、Temporal/S3/Datadog構成 |
| 2026 in LLMs — Simon Willison | 9RKrSF9d2SE | コーディングエージェント転換点、サンドボックス事件、野心 |
| The Sweet Side of the Bitter Lesson — Thariq Shihipar | xRJMiaUoD_8 | Bash/React万能批判、創造的ループの人間 |
| Ambition in AI — Jason Liu | (code2026/jason-liu.md) | 野望・オーナーシップ・審美眼 |
| Sandboxes Aren't Optional — Robert Brennan (OpenHands) | iH_wV8BbS24 | 最小権限・4つの隔離方式比較・セッション単位使い捨て環境 |
| How One Founder Manages 20 AI Coding Agents — Ryan Carson (Untangle) | BTtyguoPvBU | manager/builder構成・1ticket=1PR・レビュー3回キャップ・cron巡回・動画検証 |
| Your AI Agent Is Hiding Its Mistakes From You — Ben Holmes (Warp) | pz74r-4dGfQ | トレース採点・3種の証拠・ルーブリック・日次共通因分析→スキル修正 |
| Sub-Agents Are a Distributed System — Gish (Microsoft) | (code2026) | 部分失敗・ダーティリード・無制限ファンアウト・冪等リトライ |
| Your Agent Hates Your CLI — Ayush Bhardwaj (Allos AI) | (code2026/online-ayush-cli.md) | パイプがエラーを隠す・fail loudly・選択フラッグ・トランスクリプト監査 |
| An Agent's Change Is Not Safe Until Its Recovery Has Been Tested — Dolly & Tanmay Sah | (code2026/online-undo-button.md) | undo-bench・回復率47%・外部副作用の追跡 |
| A Skill's Instructions Are Not Its Behavior — Roey Ben Chaim (Zenity) | (code2026/online-zenity-skills.md) | スキルの爆破解析・外部ペイロード・LLM解析の欺瞞 |
| The harness has to finish the job — Joey (Harvey) | (code2026/online-harvey-agent-platform.md) | 10,000PR・ハーネス=製品・エージェント周辺こそ壊れる |
| Give the PR writer an evidence packet — Stephen Chin & Cassandra Chen | (code2026/online-stephen-cassandra.md) | PR品質・エビデンスパケット・トークン効率 |
| Who reviews the bots? — Robinhood | (code2026/online-robinhood-review.md) | 機械速度のPR vs 人間速度のレビュー |
| Green Pull Requests Can Still Break the Trunk — Uber | (code2026/online-uber-merging.md) | マージキュー・組み合わせ未検証問題 |
| Agents Made Code Free. Maintaining It Didn't Get Cheaper — Dmitry | (code2026/online-dmitry-maintenance.md) | エントロピー対策・週次スイープ・フィードバックループ |
| Software Engineering Is Becoming Factory Engineering — Zach Lloyd (Warp) | (code2026/zach-lloyd.md) | ソフトウェアファクトリー・人間スループットの天井 |
| A software factory needs pressure in both directions — Dex Horthy | (code2026/dexter-horthy.md) | Slopcode Bench・未レビューマージ30%・ファクトリー=良い工学 |
| Software Factories: Escape from Trash Mountain — Steve Yegge | (code2026/steve-yegge.md) | オーケストレーター肥大化の失敗・Bitter Lesson |
| I deleted 90% of my harness — Philipp Schmid (Google DeepMind) | (code2026/philipp-schmid.md) | モデル改善でハーネス痩身化・post-training |
| Specs have bugs too — AWS automated reasoning | (code2026/online-aws-specification-analyzer.md) | 仕様の形式検証・矛盾するルール・不明瞭プロンプトで精度20-40%低下 |
