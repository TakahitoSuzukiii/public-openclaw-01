# 週次まとめ：Claude / Claude Code による業務自動化ナレッジ（2026-07-06 号）

作成日: 2026-07-06 / STATUS: INFO / TOPIC: CLAUDEAUTO

> 注: 各記事の主張・数値・手順は出典元のものです。製品仕様やコマンドは変わり得るため、導入時は必ず一次情報で再確認してください。本記事は要約・考察であり全文転載ではありません。専門用語は綴りと意味を併記します。

## 今週のテーマ・見どころ

先週号は「Cowork による Excel/PowerPoint 資料作成」を中心に扱いました。今週は視点を変えて、**インフラ運用・SRE・CI/CD・セキュリティ運用**という「サーバ運用保守の面倒なルーティンワーク」の自動化にフォーカスします。特に、Claude Code を**アラート起点で自動起動（headless）させて障害調査〜修正 PR まで回す**「AI SRE」パターンが今週の目玉です。加えて、実運用で効いた DevOps 向け Skills の実例、CI/CD にエージェントを組み込む際のセキュリティ注意点、そして 100 以上のサブエージェントを集めた awesome シリーズを紹介します。

用語メモ:
- **SRE = Site Reliability Engineering = サイト信頼性工学**（運用をソフトウェア工学で支える考え方）
- **CI/CD = Continuous Integration / Continuous Delivery = 継続的統合／継続的デリバリー**
- **IaC = Infrastructure as Code = インフラをコードで管理する手法**
- **SLO/SLI = Service Level Objective / Indicator = サービス品質の目標値／指標**
- **Toil = トイル = 手作業で繰り返す運用作業（自動化で減らすべきもの）**
- **headless（ヘッドレス）= 対話画面なしで自動実行するモード**
- **SAST = Static Application Security Testing = 静的アプリケーションセキュリティテスト**

---

## 1. インフラ運用/SRE のルーティン自動化（今週の主役）

### 1-1. AI SRE ── オンコール信頼性ワークフロー 5 選
- 出典URL: https://www.arcade.dev/blog/claude-code-ai-sre-oncall-workflows/
- Claude Code を **headless モード（`--headless` やアラート webhook 起点）でスケジュール／自動起動**し、障害対応を回す実践パターンを 5 つ紹介。Skill にランブック（障害対応手順書）の知識を持たせ、Claude が調査・原因特定・（人の承認前提で）修正 PR 作成までを担う構成です。ポイントは「**コンパニオン方式**」──エージェントが読む・相関を取る・下書きするところまで自動化し、**マージ判断は人間が握る**こと。いきなり自動修復させず、人のトリアージ速度のボトルネックを外す設計思想が参考になります。

### 1-2. SRE 向け「インシデント対応エージェント」の作り方（公式 Cookbook）
- 出典URL: https://platform.claude.com/cookbook/managed-agents-sre-incident-responder （公式）
- 補足: https://platform.claude.com/cookbook/claude-agent-sdk-03-the-site-reliability-agent （公式・SRE エージェント）
- Anthropic 公式のレシピ集。アラート発火 → ログとランブックを読む → 根本原因を特定 → 修正 PR を開いて**承認待ちで停止**、という「読む・相関・下書き・待つ」の流れをそのまま実装できます。公式が一次情報として手順を出しているので、自社ランブックに合わせて改変する土台に最適。

### 1-3. Claude Code の可観測性を OpenTelemetry で押さえる
- 出典URL: https://generalanalysis.com/guides/claude-code-control-observability-opentelemetry
- エージェントを本番に近づけるほど「何をどのツールで実行したか」の監査が重要になります。OpenTelemetry（OTel=分散システムの計測標準）で **Metrics（集計）/ Events（個別操作ログ）/ Traces（プロンプト→モデル呼び出し→ツール実行→承認待ちの連鎖）** の 3 信号を取り、SIEM（セキュリティ監視基盤）へ流す設計を解説。`session.id` + `event.sequence` でセッションの全操作を時系列再構成でき、不審なツール呼び出しの追跡（インシデント調査）に使えます。エージェント運用の「証跡（audit trail）」を最初から設計したい人向け。

---

## 2. 実運用で効いた DevOps 向け Skills / IaC 自動化

### 2-1. 現場で実際に使っている DevOps 向け Claude Skills 8 選（Pulumi）
- 出典URL: https://www.pulumi.com/blog/top-8-claude-skills-devops-2026/
- IaC ベンダー Pulumi による「話題だけでなく実際に使っている」Skill の実録。Skill は**組織固有のやり方をモデルが呼び出せる指示セットに固める仕組み**で、汎用チャットボットを「自社の DevOps エンジニア」に近づける橋渡し、と位置づけています。何を Skill 化すると効くのかの取捨選択が具体的で参考になります。

### 2-2. Claude Code × Infrastructure as Code 実践ガイド（Spacelift）
- 出典URL: https://spacelift.io/blog/claude-code-for-infrastructure-as-code
- Terraform モジュール生成・既存構成のリファクタ・plan 出力の読解・ポリシー/テスト作成まで、IaC での使いどころを整理。ベストプラクティスとして、**ファイル変更時にスキャナ／ポリシーチェックを自動発火**させ「テスト成功をエージェントの完了条件（definition of done）にする」、**最小権限＋サンドボックス**でスコープを絞る、生成後は `terraform fmt` / `terraform validate` を自動実行して自己修正、などを提示。IaC を任せる際の「安全な回し方」が要点です。

### 2-3. DevOps エンジニアのための Claude Code（Docker/K8s/Terraform を 10 分で）
- 出典URL: https://devops.gheware.com/blog/posts/claude-code-for-devops-engineers-2026.html
- Dockerfile・Kubernetes マニフェスト・Terraform・CI/CD パイプラインを、プロジェクト文脈を踏まえて生成する入門記事。K8s では Deployment/Service/ConfigMap/Secret/Ingress/HPA/PDB/NetworkPolicy を、リソース制限・ヘルスチェック・セキュリティコンテキストといったベストプラクティス込みで生成できる、という具体例が分かりやすい。まず触ってみたい人の最初の一歩に。

---

## 3. CI/CD へのエージェント組み込みとセキュリティ運用の自動化

### 3-1.〔公式〕Claude Code GitHub Actions ── PR/Issue の `@claude` 起動
- 出典URL: https://code.claude.com/docs/en/github-actions （公式ドキュメント）
- リポジトリ: https://github.com/anthropics/claude-code-action （公式アクション）
- PR や Issue で `@claude` とメンションするだけで、コード解析・PR 作成・機能実装・バグ修正を自動化。`anthropics/claude-code-action@v1` を使えば、**CI の失敗テスト自動修正**や**インラインのコードレビュー投稿**を人間レビュー前に走らせられます。公式が土台を出しているので、自社ワークフローに配線するだけで始められます。

### 3-2.〔要注意〕エージェント時代の CI/CD セキュリティ（Microsoft）
- 出典URL: https://www.microsoft.com/en-us/security/blog/2026/06/05/securing-ci-cd-in-agentic-world-claude-code-github-action-case/
- Microsoft の脅威インテリジェンスが、**Claude Code GitHub Action が信頼できない GitHub 上のコンテンツを処理する際に CI/CD のワークフロー機密（secrets）を露出しうる**問題を指摘（Anthropic は Claude Code 2.1.128 で緩和済み）。**便利さと引き換えに攻撃面（attack surface）が増える**という重要な教訓。導入時は「バージョンを上げる」「外部からの入力を信頼しない」「secrets のスコープを最小化する」を必ずセットで。**自動化するほどセキュリティ設計が前提条件になる**という今週の裏テーマです。

### 3-3. Claude Code Security ── 推論ベースの脆弱性スキャン（DevSecOps）
- 出典URL: https://thehackernews.com/2026/02/anthropic-launches-claude-code-security.html
- 考察（SAST との違い）: https://medium.com/@instatunnel/reasoning-vs-rules-how-claude-code-security-is-disrupting-traditional-sast-0eacffd7f8bb
- 既知パターン／シグネチャの照合ではなく、**データフローを追い、コードを読み、ファイルやモジュール間の相互作用を追跡**して脆弱性を見つける「推論ベース」の SAST。PR オープン時に GitHub Actions で差分をスキャンし、ルール適用のうえ**修正案をインラインコメント**で返せます。ルールベースの従来 SAST を補完し、複数ファイルにまたがる文脈依存の欠陥を拾える点が新しい。※先週号でも触れましたが、今週は「DevSecOps パイプラインへの組み込み」観点で再整理しています。

---

## さらに試す価値（awesome シリーズ・リンク集）

- **awesome-claude-code-subagents（VoltAgent）**: https://github.com/VoltAgent/awesome-claude-code-subagents ── 100 以上の専門サブエージェント集。Core Development / Language Specialists / Infrastructure / Quality & Security など 10 カテゴリで整理され、`03-infrastructure` には `sre-engineer` / `devops-engineer` / `terraform-engineer` などがそのまま流用できる形で入っています（約 19.9k stars）。
- **awesome-agent-skills（VoltAgent）**: https://github.com/VoltAgent/awesome-agent-skills ── 公式チーム＋コミュニティ由来の 1000 以上の Skill を集約。Claude Code / Codex / Gemini CLI / Cursor 互換。
- **awesome-claude-code-toolkit（rohitg00）**: https://github.com/rohitg00/awesome-claude-code-toolkit ── エージェント・Skill・コマンド・プラグイン・hooks・MCP 設定などを幅広くまとめた総合ツールキット。
- **Runbook Generator Skill**: https://mcpmarket.com/tools/skills/operational-runbook-generator ── プロジェクトのインフラとソースを解析し、デプロイ／DB 保守／緊急時対応のランブックを自動生成する Skill。SRE の文書化トイル削減に。
- **Best Claude Code setups for observability（Claude Directory）**: https://www.claudedirectory.org/for/observability ── 可観測性・監視向けの `.claude/` 構成例のまとめ。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
