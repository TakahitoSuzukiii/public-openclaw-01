作成日: 2026-08-03 / STATUS: INFO / TOPIC: CLAUDEAUTO

# 週次まとめ：Claude / Claude Code による業務自動化ナレッジ（2026-08-03 号）

> 注: 各記事の主張・数値・手順は出典元のものです。製品仕様やコマンドは変わり得るため、導入時は必ず一次情報で再確認してください。本記事は要約・考察であり全文転載ではありません。専門用語は綴りと意味を併記します。

## 今週のテーマ・見どころ

先週（7/26 号）は「人の**判断**そのものを Skill として定義し、運用フローに埋め込む」ことを主役にしました。今週は視点を変えて、**「ターミナルに常駐させたエージェントに、インフラの日常オペレーションそのものを委譲する」実装レベルの話**を軸にします。焦点は次の4系統です。

- **インフラ日常業務の委譲** ── `kubectl`・`terraform`・ログ解析を、パイプと Plan Mode（計画モード）で安全に AI へ渡す実践。
- **セキュリティ運用の自動化（防御側）** ── Anthropic 公式の「セキュリティレビュー用 GitHub Action」で、PR（プルリクエスト）の変更差分に潜む脆弱性を自動指摘する。
- **CI/CD と Playwright による E2E 自動化** ── GitHub Actions / GitLab / Jenkins への組み込みと、テスト生成〜失敗トリアージ。
- **Agent Skills エコシステムの整理** ── 仕様が `agentskills.io` に集約され、公式スキル集・awesome 系がハブ化してきた現状。

加えて優先トピックである **Excel/PowerPoint レポート自動生成の続報**を挟み、末尾に awesome 系リンク集を添えます。

用語補足: SRE=Site Reliability Engineering＝サイト信頼性工学（運用を工学的に扱う職種）、CI/CD=Continuous Integration / Continuous Delivery＝継続的統合／継続的デリバリー、IaC=Infrastructure as Code＝コードで基盤を定義する手法、E2E=End to End＝利用者操作を端から端まで通す試験、PR=Pull Request＝変更提案、MCP=Model Context Protocol＝AI と外部ツールをつなぐ標準規格。

---

## 1. インフラ運用 ── 「日常オペレーション」をターミナル常駐エージェントに委譲する（今週の主役）

「自分はコードを書くエンジニアじゃないから AI コーディングは関係ない」は誤解、というのが今週の出発点です。Claude Code の本質は**コードを書く能力ではなく、ターミナルでコマンドを自律実行し、ログを読み、状態を診断する能力**にある、という整理が現場感に合っています。

### 1-1.〔実践ガイド〕kubectl・terraform・ログ解析をパイプと Plan Mode で自動化
- 出典URL: https://techotakulab.com/claude-code-for-infra-engineers-2026/
- インフラ担当が Claude Code を「諜報エージェント」として使う実践ガイド。核心は3点。①**パイプ活用** ── `kubectl logs deployment/my-app --tail=200 | claude -p "エラーの根本原因を特定して"` のように、長大なログを手で読まず標準入力で流し込んで分析させる。②**Plan Mode（計画モード）** ── `Shift+Tab` で切り替えると Claude は**読み込みのみで一切変更しない**。Terraform リファクタリング前に「影響範囲・変更手順・触らない箇所（Non-goals）」を Markdown 計画書として出させ、人間が承認してから実装モードへ移す。「まず調べるだけ、手は出すな」というインフラ屋の慎重さをそのまま仕組みにできる。③**CLAUDE.md で安全ルール** ── 毎回読み込まれる恒久設定に、実行してよいコマンド範囲や本番系への禁止事項を明記してから本格運用へ。深夜の Kubernetes 障害を `get nodes → describe node → get events` と自律調査させる例が具体的です。

### 1-2.〔事例集〕実装プロンプト付きの業務自動化10事例
- 出典URL: https://uravation.com/media/claude-code-automation-case-studies-2026/
- 請求書・メール・レポート・帳票・議事録・契約書・月次集計・週報・データ照合・クレンジングの10業務を、**実装プロンプト付き**で公開したまとめ。運用現場でも数の多い「月次集計」「週報」「データ照合」あたりは、そのまま自分の定型作業に当てはめやすい。結論として「効果が出る企業は例外なく **“工数を時間単位で測れる定型業務”** から着手していた」という指摘は、導入の順番を考えるうえで実務的です（数値・ROI は出典元の主張のため自環境で検証を）。

---

## 2. セキュリティ運用の自動化（優先トピック・防御側）── 変更差分の脆弱性を PR で自動指摘

ゼロデイ対策の第一歩は「脆弱なコードを本番に**入れない**」こと。今週は Anthropic 公式が出している、**PR の差分だけを対象に脆弱性を自動レビューする GitHub Action** を取り上げます。防御的自動化の具体例として実装がそのまま参考になります。

### 2-1.〔公式〕`anthropics/claude-code-security-review` ── 差分アウェアなセキュリティレビュー
- 出典URL: https://github.com/anthropics/claude-code-security-review
- 解説ブログ: https://www.anthropic.com/news/automate-security-reviews-with-claude-code
- Claude の意味理解（semantic analysis）で、パターンマッチを超えて脆弱性を検出する GitHub Action。特徴は、①**Diff-Aware Scanning** ── PR では変更ファイルのみを解析し無駄を省く、②**PR 自動コメント** ── 発見事項をそのまま PR に書き込む、③**False Positive Filtering** ── 誤検知を抑えて「本物の脆弱性」に集中、④**言語非依存**。導入は `.github/workflows/security.yml` に数行を足すだけ（`permissions` は `pull-requests: write` と `contents: read`、API キーは GitHub Secrets 経由）。
- ⚠️ **重要な注意（必読）**: 公式は「この Action は**プロンプトインジェクション攻撃に対して堅牢化されていない**ので、**信頼できる PR にのみ使うこと**」と明記。公開リポジトリでは GitHub の「Require approval for all external contributors（外部貢献者のワークフローに承認を必須化）」を有効にし、メンテナがレビューした後にのみ実行されるようにするのが前提です。便利さと引き換えに、**外部からの入力を無条件に信用しない**設計が必須という好例。

---

## 3. CI/CD と Playwright ── パイプラインに載せる E2E 自動化

Claude Code をローカル補助だけで使うのは半分もったいない、という潮流は今週も続きます。今週は特に **Playwright（ブラウザ自動操作フレームワーク）による E2E テスト**を CI に載せる話を厚めに。

### 3-1.〔ガイド〕CI/CD 統合 ── GitHub Actions / GitLab CI / Jenkins の例つき
- 出典URL: https://crazyrouter.com/en/blog/claude-code-ci-cd-integration-guide-2026
- Claude Code をパイプラインに組み込み、コードレビュー・テスト生成・デプロイ検証を自動化する統合ガイド。GitHub Actions だけでなく **GitLab CI・Jenkins** の設定例まで揃うのが実務向き。自社の CI が GitHub 以外でも移植しやすい。

### 3-2.〔実践〕テスト自動化 ── Playwright・Jira・Postgres の MCP を配線し、失敗をトリアージ
- 出典URL: https://qaskills.sh/blog/claude-code-test-automation-guide
- Playwright / Jira / Postgres の MCP サーバを Claude Code に接続し、**実行可能なテストを生成 → 失敗を自動トリアージ**するまでの実践。テストを「書く」だけでなく「落ちた原因を切り分ける」ところまで含めるのが今どきの構成。E2E に踏み込むなら、Playwright/Cypress と CI の落とし穴を整理した記事（https://claudecode-lab.com/en/blog/claude-code-e2e-testing/ ）も併読を。

---

## 4. Excel / PowerPoint レポート自動生成（優先トピック・続報）

インフラ運用でも数の多い「報告資料づくり」。今週は**エンドツーエンドの実践デモ**を1本。

### 4-1.〔デモ〕ニュース収集 → Excel → PowerPoint → Slack を一気通貫
- 出典URL: https://genai-ai.co.jp/ai-kanri/blog/cc-yt-automation-masterclass-88/
- 「情報収集 → Excel 集計 → PowerPoint 資料化 → Slack 共有」までを一連のワークフローとして Claude Code に組ませる実演。ポイントは **CLAUDE.md にナレッジを蓄積して精度を上げ続ける**という運用思想。単発の資料生成で終わらせず、「毎週回る仕組み」に育てる発想は、当ノートの週次運用とも相性が良いです（本記事自体、同じ考え方で自動生成しています）。

---

## 5. さらに試す価値（awesome / エコシステム・リンク集）

Agent Skills は 2026 年前半で仕様が固まり、参照先が集約されてきました（`skills.sh` は `agentskills.io` へリダイレクト）。まずここから当たると全体像がつかめます。

- **Anthropic 公式スキル集** ── https://github.com/anthropics/skills （Excel/PowerPoint/Word/PDF など公式プリビルトの原典）
- **Agent Skills 仕様（オープン標準）** ── https://agentskills.io
- **awesome-claude-code** ── https://github.com/hesreallyhim/awesome-claude-code （スキル・サブエージェント・ステータスライン・プラグイン・開発ツールの厳選集）
- **Agent Skills for SRE/DevOps（解説＋Azure Terraform × Defender スキル実例）** ── https://yisusvii.github.io/posts/claude-code-codex-skills-devops-sre-cloud-2026/
- **デプロイ/CI-CD 向けセットアップ集** ── https://www.claudedirectory.org/for/deployment
- **Claude Code 設定ガイド（エージェント自身が実行できる形式）** ── https://github.com/AlexandrG539/claude-code-setup-guide

> 今週の実務的な学び: 「**読むだけ（Plan Mode）→ 人が承認 → 実行**」「**外部入力を信用しない（security-review の注意書き）**」の2つは、AI にオペレーションを委ねるときの安全設計の型として、そのまま自分の運用ルールに転用できます。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
