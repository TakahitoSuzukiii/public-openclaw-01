# Claude 業務自動化ナレッジ（週次）

> 作成日: 2026-06-10 / STATUS: INFO / TOPIC: CLAUDEAUTO

## 今週のテーマ / 見どころ

今週は「**資料作成の自動化**」と「**セキュリティ運用（特にゼロデイ対応）の自動化**」を主軸に据えました。Excel/PowerPoint といった“面倒だが避けられない”定型業務を Claude + MCP（Model Context Protocol = AIと外部ツールをつなぐ標準プロトコル）でどう削るか、という実務目線の記事を中心に集めています。

加えて、2026年に話題となった「AIがゼロデイ脆弱性を大量発見し、人間のトリアージ（=対応の優先順位付け）が追いつかない」という構造問題と、その逆手を取った**SOC（Security Operations Center = セキュリティ監視チーム）のアラート自動仕分け**の事例も取り上げます。最後に SRE/CI/CD の運用自動化、ベストプラクティス、awesome 系リンク集を添えます。

注: 各記事の主張・数値は出典元のものです。製品仕様や数値は変わりうるため、導入時は必ず一次情報で再確認してください。

---

## 1. インフラ運用の面倒なルーティン自動化（PPT / Excel レポート）

最優先トピック。手作業の温床である「Excel集計 → PowerPoint化 → 定例レポート」を、Claude にどこまで任せられるかという実例です。

### 1-1. Claude Code + MCP で PowerPoint を自動生成
- 出典: [PowerPoint手作業終了の未来が見えた！Claude Code + MCPでプレゼン資料を自動生成する (Zenn / acntechjp)](https://zenn.dev/acntechjp/articles/69c7340679b4b1)
- python-pptx（Pythonで .pptx を操作するライブラリ）ベースの MCP サーバーを介し、Claude Code にスライド生成・編集を任せる構成。テンプレートのレイアウト・配色を踏襲したままスライドを量産できる点が実務向き。
- 適用シーン: 定例の進捗報告・構成図入り提案書。注意点として、図形配置の細部は人手の最終調整が前提。

### 1-2. Excel データを 1 分で「パワポ化」
- 出典: [Excelデータを1分でパワポ化！Claude連携で資料作成を自動化する神ワザ (あなたのAI顧問)](https://ai-advisors.jp/media/ai-news/claude-excel-powerpoint/)
- 集計済み Excel を入力に、章立て・グラフ・要約スライドまで一気通貫で生成する流れを解説。月次・週次レポートの“作り直し”コストを削るのが狙い。

### 1-3. Claude Skills で Excel / PowerPoint / PDF を生成
- 出典: [Claude Skillsを使ってExcel・PowerPoint・PDFを自動生成する (Zenn / ttks)](https://zenn.dev/ttks/articles/963d86a67a2083)
- Claude の「Skills」機能（特定作業の手順をパッケージ化する仕組み）で、Office 系ファイルをコードから生成する方法。API 経由で定型成果物を量産する用途に向く。

### 1-4. 公式: Claude for PowerPoint アドイン
- 出典: [Use Claude for PowerPoint (Claude 公式ヘルプ)](https://support.claude.com/en/articles/13521390-use-claude-for-powerpoint) / [試用レビュー (NEXTSCAPE blog)](https://blog.nextscape.net/archives/2026/03/17/094853)
- PowerPoint に Claude を組み込む公式アドイン。デッキのスライドマスター・レイアウト・フォント・配色を読み取り、ブランドを崩さずに生成・編集する設計。テンプレート準拠を保てるのが企業利用での要点。

### 1-5. CRM データから営業レポートを自動生成
- 出典: [Claude Codeで営業レポートを自動生成する方法｜HubSpot CRMデータの分析から資料作成まで (start-link.jp)](https://start-link.jp/hubspot-ai/ai/claude-code-practice/claude-code-sales-report-automation)
- HubSpot（CRM = 顧客関係管理ツール）の MCP から当月の取引データを取得し、`/monthly-report` の一発で同品質のレポートを毎月生成する例。データ取得→分析→資料化を1つのワークフローに畳む発想が参考になる。

---

## 2. ゼロデイ攻撃対策・セキュリティ運用の自動化

“AIが攻める側にも守る側にも回る”2026年の現実。発見と防御の両面を押さえます。

### 2-1. AIによるゼロデイ大量発見と「トリアージ崩壊」問題
- 出典: [Claude Found 500 Zero-Days. Who Patches Them Before Attackers Arrive? (Futurum)](https://futurumgroup.com/insights/claude-found-500-zero-days-who-patches-them-before-attackers-arrive/) / [Anthropic Rolls Out Claude Security for AI Vulnerability Scanning (Infosecurity Magazine)](https://www.infosecurity-magazine.com/news/anthropic-claude-security-for-ai/)
- Anthropic の Frontier Red Team が、本番OSS（オープンソースソフトウェア）から500件超の高深刻度脆弱性を発見・検証したと報告。一方で上流に修正が取り込まれたのはごく一部で、「**発見速度に人間のトリアージ・パッチ適用が追いつかない**」という新たな攻撃面が生まれている、という論点。
- 示唆: 発見だけでなく「自環境に効くかの優先度付け」と「修正フローの自動化」をセットで設計しないと、発見が逆にリスクになる。

### 2-2. SOC のアラート自動トリアージ（実務で最も効く守りの自動化）
- 出典: [AI Cybersecurity in 2026 (MindStudio)](https://www.mindstudio.ai/blog/ai-cybersecurity-2026-claude-mythos-zero-day-exploits)
- 現実的に効果が出ている守りの用途は「SOCのアラート一次仕分け（誤検知の削減）」「脆弱性パターンの自動コードレビュー」「自社環境に照らした CVE 深刻度評価」「脅威インテリジェンスの付加情報付け」。
- 推奨パターン: エージェントが CVE フィードを監視 → 詳細取得 → 自社の技術資産台帳と突合 → 文脈に応じて深刻度評価 → 優先度付き要約を Slack に投稿。**“発見”より“自社にとっての意味付け”を自動化**するのが肝。

### 2-3. ⚠️ 自動化の落とし穴: Claude Code GitHub Action のシークレット漏えい
- 出典: [Securing CI/CD in an agentic world (Microsoft Security Blog)](https://www.microsoft.com/en-us/security/blog/2026/06/05/securing-ci-cd-in-agentic-world-claude-code-github-action-case/) / [Claude Code GitHub Action Flaw (The Hacker News)](https://thehackernews.com/2026/06/claude-code-github-action-flaw-let-one.html)
- Microsoft Threat Intelligence が、Claude Code の GitHub Action が**信頼できない入力（Issue本文・PR説明・コメント等）を処理する際に CI/CD のシークレットを露出しうる**問題を指摘。1件の悪意ある Issue でリポジトリ乗っ取りに繋がる経路が報告された。
- 教訓: エージェントに外部入力を渡すワークフローは、プロンプトインジェクション（=入力に紛れた不正命令）前提で権限を絞る。シークレットへのアクセス範囲・トリガー条件・承認ゲートの見直しを。

---

## 3. インフラ / クラウド / SRE / CI/CD / Playwright 自動化

### 3-1. Claude Code で IaC（Infrastructure as Code）— Terraform 実例
- 出典: [Claude Codeではじめる、インフラ構築/運用 ── IaC編 (Zenn / nocall)](https://zenn.dev/nocall/articles/09534c8b6832a2)
- AWS インフラを Claude Code と共に Terraform 化し、短期間で大量のコードを追加した事例。**staging は `terraform apply` まで任せ、production は `plan` 止まりで人が承認実行**という権限分離が実務的で参考になる。AGENTS.md の HITL（Human-in-the-Loop = 人間が承認に介在する）原則とも整合。

### 3-2. DevOps で実際に使う Claude Skills 8選
- 出典: [The Claude Skills I Actually Use for DevOps (Pulumi Blog)](https://www.pulumi.com/blog/top-8-claude-skills-devops-2026/)
- SLO/SLI（サービスレベル目標/指標）定義、エラーバジェット計算、ゴールデンシグナルのダッシュボード、トイル（=繰り返しの手作業）削減など、SRE 観点で“本当に使う”スキルを厳選紹介。Prometheus/Grafana の設定や復旧 Runbook の生成まで踏み込む。

### 3-3. Claude Code Routines — 無人でCI/CDを回す
- 出典: [Claude Code Routines: Autonomous Dev Automation (AI Automation Global)](https://aiautomationglobal.com/blog/claude-code-routines-automated-dev-workflows-2026)
- プロンプト＋連携リポジトリ＋トリガー（スケジュール / API / GitHubイベント）を設定し、ローカルマシンや人の常駐なしでタスクを自動実行する仕組み。定例運用・夜間バッチ的なエージェント運用に向くが、§2-3 のセキュリティ配慮が前提。

---

## 4. ベストプラクティス / グッドパターン

### 4-1. Hooks / Subagents / Skills の使い分け
- 出典: [Claude Code Advanced Best Practices (SmartScope)](https://smartscope.blog/en/generative-ai/claude/claude-code-best-practices-advanced-2026/) / [公式 Hooks ガイド](https://code.claude.com/docs/en/hooks-guide)
- 要点:
  - **Hooks は“100%強制”したい決定的処理に使う**（lint・テスト・セキュリティスキャン）。「停止前にテスト実行」「生成ファイルの編集をブロック」「依存変更後にスキャン」など。
  - **Subagents は文脈を分離したい時**。冗長なログ探索を別コンテキストに隔離し、要約だけを親に返す。ただしサブエージェントは親のスキルを継承しないため明示プリロードが必要。
  - トレードオフは Skills（同一文脈・低コスト）→ Subagents（文脈分離）→ Agent Teams（別プロセス）。右に行くほど分離・並列が増えるが、オーバーヘッドも増える。

### 4-2. CLAUDE.md の設計（“育てる設定ファイル”）
- 出典: [Designing CLAUDE.md correctly (obviousworks)](https://www.obviousworks.ch/en/designing-claude-md-right-the-2026-architecture-that-finally-makes-claude-code-work/)
- 「**200行以内に保つ**（長すぎると無視される）」「月次で更新する生きた文書として扱う」「Claude がミスをするたびにルールを1つ足す」。NEXUS の AGENTS.md 運用方針にもそのまま効く知見。

---

## 5. さらに試す価値 — awesome シリーズ / リンク集

- [hesreallyhim/awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) — 手キュレーションの定番リスト。スキル・フック・スラッシュコマンド・オーケストレーターを網羅。
- [rohitg00/awesome-claude-code-toolkit](https://github.com/rohitg00/awesome-claude-code-toolkit) — エージェント/スキル/コマンド/プラグイン/MCP設定を大量収録した“全部入り”系。
- [FlorianBruniaux/claude-code-ultimate-guide](https://github.com/FlorianBruniaux/claude-code-ultimate-guide) — 初心者〜パワーユーザー向けの体系ガイド。チートシート付き。
- [Microsoft Office (PowerPoint & Excel) MCP Server](https://mcpservers.org/servers/jenstangen1/pptx-xlsx-mcp) — §1 の PPT/Excel 自動化を試す際の MCP サーバー候補。
- [Claude Code Subagents: A 2026 Practical Guide (Tembo.io)](https://www.tembo.io/blog/claude-code-subagents) — サブエージェント設計の実践ガイド。

> ※ awesome 系のスター数や収録数は時期により変動します。利用時は各リポジトリのライセンス・更新日を確認のうえ取り込んでください。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
