# Claude 業務自動化ナレッジ（週次）

作成日: 2026-06-15 / STATUS: INFO / TOPIC: CLAUDEAUTO

## 今週のテーマ / 見どころ

先週は「PPT / Excel 資料の自動生成」と「ゼロデイ発見」を中心に扱いました。今週はあえて軸をずらし、**SRE（Site Reliability Engineering＝サイト信頼性工学）とオンコール運用の自動化**を主役に据えます。深夜にアラートが鳴ってから、ログを集め、ランブック（runbook＝障害対応手順書）を探し、原因を特定し、修正 PR（Pull Request＝変更提案）を出すまで——この「コードを書く前の外側のループ」を Claude にどこまで任せられるか、というのが今週の通底テーマです。

あわせて、インフラエンジニアの日常コマンド作業（kubectl / Terraform / ログ解析）の委譲、そして「エージェントに本番環境への書き込みを許す」ときのセキュリティ設計（サンドボックス・コマンド許可リスト・人間承認ゲート）も取り上げます。

> 注: 各記事の主張・数値・手順は出典元のものです。製品仕様やコマンドは変わりうるため、導入時は必ず一次情報で再確認してください。本記事は要約・考察であり、全文転載ではありません。

---

## 1. インフラ運用の面倒なルーティンを委譲する（kubectl / Terraform / ログ解析）

### 1-1. インフラエンジニアのための Claude Code 実践ガイド
- 出典: [【2026年版】インフラエンジニアのためのClaude Code実践ガイド：kubectl・terraform・ログ解析を自動化する (techotakulab)](https://techotakulab.com/claude-code-for-infra-engineers-2026/)
- 「Claude Code はコードを書くツールではなく、ターミナルに常駐してコマンドを自律実行し、ログを読み、インフラ状態を診断するエージェント」という捉え方を提示。Kubernetes ノード障害の調査（`kubectl get nodes` → `describe` → `events` → ログ確認 → 原因特定）を AI に肩代わりさせる流れを具体的に解説します。
- 適用シーン: VPS / WSL2 を含む環境での導入手順、`CLAUDE.md` での安全ルール定義、まず `kubectl logs | claude -p` から小さく試す、という現場目線の入り方が参考になります。

### 1-2. AWS インフラエンジニアのための Claude Code 完全活用ガイド
- 出典: [AWSインフラエンジニアのための Claude Code 完全活用ガイド 2026 — Context 肥大化を制する者がAI開発を制す (Qiita)](https://qiita.com/hidetarou2013@github/items/0bdcbca4efa29d9dd28e)
- CDK（Cloud Development Kit＝コードで AWS 構成を書く仕組み）スタック設計から、`cdk-nag` を使ったセキュリティレビュー、PR 作成、ドキュメント化までを「一人のシニアエンジニア」として回す活用法。
- 注意点: 記事のキモは **Context（文脈）肥大化の制御**。長い対話で AI の精度が落ちる問題への対処として、タスク分割やコンテキスト管理を重視しています。インフラの大規模リポジトリほど効いてくる観点です。

---

## 2. SRE / オンコール運用の自動化 ★今週の主役

### 2-1. 公式: SRE インシデント対応エージェント（Claude Managed Agents）
- 出典: [Build an SRE incident response agent with Claude Managed Agents (Claude 公式 Cookbook)](https://platform.claude.com/cookbook/managed-agents-sre-incident-responder)
- アラート発火（PagerDuty Webhook を想定）を起点に、エージェントがログとランブックを読み、原因を特定し、修正 PR を作成して**マージ前に人間の承認を待つ**——という一連を組む公式チュートリアル。Skill でチーム独自のランブック規約を教え、Console（管理画面）が全ステップを自動記録します。
- 注目点: 「破壊的アクション（マージ）は人間承認ゲートの背後に置く」という設計が明示されている点。HITL（Human-in-the-Loop＝人間が判断ループに介在する仕組み）を前提にした堅実な作りです。

### 2-2. 公式: サイト信頼性エージェント（Claude Agent SDK + MCP）
- 出典: [The site reliability agent (Claude 公式 Cookbook)](https://platform.claude.com/cookbook/claude-agent-sdk-03-the-site-reliability-agent)
- 2-1 が「読んで PR を出す」までなら、こちらは一歩進めて**設定ファイルの編集やサービス再起動まで実行**するエージェント。MCP（Model Context Protocol＝AI に外部ツールをつなぐ標準）ツールを、ディレクトリ制限・コマンド許可リスト（allowlist）・検証フックで囲って「安全な書き込み権限」を与える設計を学べます。
- 学び: 「凝ったプロンプトより、明確なツール説明文のほうが自律動作を正しく駆動する」という指摘が実務的。メトリクス・ログ・アラートを横断して根本原因を合成し、ポストモーテム（postmortem＝事後分析）まで残します。

### 2-3. Claude Code で回す 5 つのオンコール・ワークフロー
- 出典: [AI SRE with Claude Code: 5 On-Call Reliability Workflows (Arcade.dev)](https://www.arcade.dev/blog/claude-code-ai-sre-oncall-workflows/)
- オンコールが本題に入る前に払う「コンテキスト読み込み税」——Datadog→CI→Jira→Slack を行き来して仮説を立てるまでの 8 分——を問題提起。インシデント・トリアージ、ランブック実行、ポストモーテム、SLO（Service Level Objective＝サービス品質目標）調査、オンコール引き継ぎの 5 業務を Claude Code から回す構成を解説します。
- 勘所: 「モデルの賢さ」ではなく「チーム横断・本番接続・認証/スコープ/監査を担保する実行レイヤー」がボトルネックだ、という整理が鋭い指摘です。

### 2-4. Claude Skills でインシデント対応を仕組み化する
- 出典: [Mastering Incident Response with Claude Skills: Runbooks, Postmortems, and On-Call Automation (claudeskills.info)](https://claudeskills.info/blog/mastering-incident-response-claude-skills/)
- ランブック・ポストモーテム・オンコール引き継ぎ・GitOps の 4 つを Skill ファイルとして `.claude/commands/` に置き、「正しい手順を“最も抵抗の少ない道”にする」アプローチ。障害時に手順書が 3 クリック先にあるから使われない、という現場あるあるへの処方箋です。
- 適用シーン: ランブックが Notion、テンプレが Confluence……と散らばっているチームの標準化に効きます。

---

## 3. 自動化に「本番への書き込み権限」を与えるときのセキュリティ

優先度の高いセキュリティ運用の観点として、今週は「攻撃検知」よりも**“自律エージェントに権限を渡す”側のリスク管理**を取り上げます（先週はゼロデイ大量発見と CI/CD シークレット漏えいを扱いました）。

### 3-1. 安全な書き込み権限の設計パターン（公式 Cookbook より）
- 出典: 上記 [2-2 The site reliability agent](https://platform.claude.com/cookbook/claude-agent-sdk-03-the-site-reliability-agent) / [2-1 Managed Agents SRE responder](https://platform.claude.com/cookbook/managed-agents-sre-incident-responder)
- エージェントに本番操作を許す際の実装上の勘所がそのままセキュリティ統制になります: ①書き込み先ディレクトリの制限、②実行可能コマンドの許可リスト化、③検証フックでの事前チェック、④破壊的操作（マージ・再起動）の人間承認ゲート、⑤Console による全操作の監査ログ化。
- 注意点: 「自律 remediation（自動修復）」は便利な反面、許可リストや承認ゲートが緩いと事故が本番に直行します。導入は確信度の低いアクションを必ず人間にエスカレーションする設計から。

### 3-2. 「完全自律」へ進む潮流と、そのリスクの所在
- 出典: [AI Meets SRE in 2026: Autonomous Operations, New Tools, and What to Learn Next (yisusvii)](https://yisusvii.github.io/posts/ai-sre-news-2026/)
- 2026 年は「AI 支援のインシデント対応」が「自律インシデント対応」へと言葉が変わった年、と総括。検知→トリアージ→相関→（既知障害クラスは）自動修復→（確信度が閾値未満なら）人間へエスカレーション、という標準パターンを提示します。OOM kill・Pod 退避・DB コネクション枯渇などを「事前承認済みランブックで自動実行」する範囲が広がっているとのこと。
- 考察: 自動修復の対象を「既知の失敗クラス」に限定し、確信度の閾値で人間に渡す——この線引きこそがセキュリティと信頼性の境界線になります。

---

## 4. ベストプラクティス / グッドパターン

### 4-1. インシデント対応は「構造」がないと失敗する
- 出典: 上記 [2-4 claudeskills.info](https://claudeskills.info/blog/mastering-incident-response-claude-skills/)
- 「手順書もテンプレもあるのに、プレッシャー下では誰も一貫して使えない」——だから手順を Skill として固定化し、呼び出しを 1 行にする。プロセスを増やすのではなく、正しいプロセスを最短経路にする、という発想が良パターンです。

### 4-2. コンテキスト管理を運用の前提に置く
- 出典: 上記 [1-2 Qiita AWS ガイド](https://qiita.com/hidetarou2013@github/items/0bdcbca4efa29d9dd28e)
- 大規模インフラリポジトリでは、長い対話による Context 肥大化が精度低下の主因になります。タスク分割・スコープ限定・`CLAUDE.md` での前提固定をセットで運用するのが定石です。

---

## 5. さらに試す価値 — リンク集 / Skill・Plugin

- [Claude 公式 Cookbook（エージェント実装レシピ集）](https://platform.claude.com/cookbook) — SRE 以外にも observability エージェント等が揃う一次情報。まずはここから。
- [harrylowkey/claude-code-plugins (GitHub)](https://github.com/harrylowkey/claude-code-plugins) — incident-response プラグインなど、運用系の Claude Code プラグイン集。
- [Incident Response Runbook Generator (mcpmarket)](https://mcpmarket.com/tools/skills/incident-response-runbook-generator) — SEV 定義・シナリオ別修正・ロールバック手順を含むランブックを生成する Skill。
- [Incident Response / Reliability Engineering Skill (mcpmarket)](https://mcpmarket.com/tools/skills/incident-response-reliability-engineering) — ブレームレス・ポストモーテムやカオスエンジニアリング演習までカバーする Skill。
- [Claude Agent SDK for Python (GitHub)](https://github.com/anthropics/claude-agent-sdk-python) — 2-2 で使われる公式 SDK。MCP ツールを安全に縛って組む実装の出発点。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
