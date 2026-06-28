作成日: 2026-06-29 / STATUS: INFO / TOPIC: CLAUDEAUTO

# 週次まとめ：Claude / Claude Code による業務自動化ナレッジ（2026-06-29 号）

## 今週のテーマ・見どころ

今週は「**実際にインフラを触る自動化**」と「**セキュリティ運用の守り**」に軸を置きました。前週（PowerPoint/Excel 資料作成・Playwright 中心）とは別ジャンルです。具体的には、

- IaC（Infrastructure as Code＝インフラをコードで定義・管理する手法）を Claude Code で書く実務事例（Terraform / AWS CDK）。
- ゼロデイ・脆弱性対応を回す「セキュリティ運用」系の Skill / MCP（Model Context Protocol＝AI と外部ツールをつなぐ標準プロトコル）。
- SRE（Site Reliability Engineering＝サイト信頼性工学。システムの安定運用を工学的に行う考え方）のオンコール・インシデント対応をエージェント化する公式レシピ。
- そして「自動化する側」が逆に攻撃される **Claude Code 自体のセキュリティリスク**にも触れます。

共通する勘所は **「段階的な権限委譲＋人間の最終承認（HITL）」**。`apply` や `merge` は人が承認する設計が、各事例で繰り返し強調されています。

---

## 1. ITインフラ運用のルーティン自動化（IaC・サーバ構築）

### 1-1. Claude Code で AWS インフラを Terraform 化（5日で約1万行）
- 出典URL: https://zenn.dev/nocall/articles/09534c8b6832a2
- 音声AIプラットフォーム運営チームが、既存 AWS インフラを Claude Code と一緒に Terraform 化した事例です。5日間で 220 ファイル・約 10,000 行を追加。ポイントは権限設計で、**staging 環境は `terraform apply` まで Claude に任せ、production は `plan` までに制限して apply は人間が承認**しています。「どこまで AI に委ねるか」を環境ごとに線引きする好例で、自分の現場の権限境界を考える材料になります。

### 1-2. Terraform 未経験の運用エンジニアが Claude Code で構築
- 出典URL: https://quo-digital.hatenablog.com/entry/2026/02/05/144740
- Terraform 未経験の運用エンジニアが、Claude Code を相棒に AWS 環境を構築した記録です。マージ前に Claude Code の `/self-review`（自己レビュー）コマンドを使い、人が見落としがちな構文ミスや命名規則の揺れをチェックさせています。**「書かせる」だけでなく「レビューさせる」までを一連の自動化に組み込む**のが実務的で、学習コストを下げつつ品質を担保したい運用チームの参考になります。

### 1-3. インフラ経験ほぼ0のエンジニアが AWS CDK でIaC
- 出典URL: https://tech.dentsusoken.com/entry/2026/03/09/
- インフラ経験がほぼ無いエンジニアが、ECS＋Aurora 構成を AWS CDK（コードでAWSリソースを定義する公式ツール）で組んだ事例です。結論として「**設計者兼レビュアとして関われば、本番に持っていけるレベルのコードが生成できる**」と報告。AI に丸投げするのではなく、人間が設計意図を持ちレビューする前提なら戦力になる、という現実的な温度感が参考になります。

---

## 2. ゼロデイ攻撃対策・セキュリティ運用の自動化

### 2-1. CVE MCP Server ── Claude Code を「27ツールのセキュリティアナリスト」に
- 出典URL: https://qiita.com/kai_kou/items/cdbfb6155c4a84a14d65
- CVE（Common Vulnerabilities and Exposures＝共通脆弱性識別子）情報の調査を、27 ツール・21 データソースを1つに束ねた MCP サーバー経由で Claude Code から扱えるようにするハンズオンです。脆弱性の調査・トリアージ（優先度づけ）を会話で回せるため、**DevSecOps（開発・セキュリティ・運用を一体化する考え方）のルーティン調査**を効率化できます。手元の脆弱性対応フローに組み込む価値があります。

### 2-2. security-guidance プラグイン ── 生成コードをリアルタイム審査
- 出典URL: https://qiita.com/kai_kou/items/411e39ef6571379ec369
- Claude Code が生成したコードをその場で審査し、脆弱性を検知したら**同一セッション内で修正まで促す**公式プラグインの入門記事です。「危ないコードを書いてしまう」リスクを生成の瞬間に潰せるため、セキュアコーディングの底上げに有効。自動化でコード量が増えるほど効いてくる守りの仕組みです。

### 2-3. サイバーセキュリティ向け Claude Code Skill 集（15種）
- 出典URL: https://github.com/Masriyan/Claude-Code-CyberSecurity-Skill
- 攻撃（オフェンシブ）・防御（ディフェンシブ）・リバースエンジニアリング・脅威ハンティング・CSOC（Cyber Security Operations Center＝セキュリティ監視運用センター）自動化などを 15 個の Skill にまとめた OSS コレクションです。**自社のSecOps（セキュリティ運用）手順を Skill 化する際のテンプレート**として参考になります。導入時は後述の「Skill の安全性」に必ず注意してください。

### 2-4.〔注意喚起〕Claude Code / MCP を安全に使うための実践ガイド
- 出典URL: https://zenn.dev/ytksato/articles/057dc7c981d304
- 「自動化する側」が狙われる話です。悪意ある MCP サーバーへ接続するだけで OS コマンドインジェクションが成立し得る RCE（Remote Code Execution＝遠隔コード実行）脆弱性（例：mcp-remote 系）など、実被害事例から学ぶ安全ガイド。**最大のリスクは「古いバージョンを使い続けること」**と整理されています。MCP を増やすほど攻撃面も広がるため、出所不明の MCP / Skill を入れない・バージョンを上げ続ける、を徹底しましょう。関連して、Snyk が 2026年2月に公開Skillを 3,984 件スキャンしたところ **13.4% に重大な脆弱性、76 件に悪意あるペイロード**が見つかった、という調査もあります（出典: https://www.pulumi.com/blog/top-8-claude-skills-devops-2026/ ）。

---

## 3. SRE・CI/CD・インシデント対応の自動化

### 3-1.〔公式〕Claude で SRE インシデント対応エージェントを作る
- 出典URL: https://platform.claude.com/cookbook/managed-agents-sre-incident-responder
- Anthropic 公式 Cookbook のレシピ。PagerDuty（インシデント通知サービス）のアラートを受け取り、**ログから障害シグネチャを特定 → インフラリポジトリから根本原因を探索 → 修正PRを作成 → 承認を要求し、承認された場合のみマージ**という一連の流れを組みます。Skill でチームの runbook（障害対応手順書）規約を教え込み、bash / edit はサンドボックスで実行。「自動で直すが、マージは人が承認」という安全設計の手本です。

### 3-2. AI SRE：オンコール信頼性ワークフロー5選
- 出典URL: https://www.arcade.dev/blog/claude-code-ai-sre-oncall-workflows/
- Claude Code を使ったオンコール（障害待機当番）向けの実用ワークフロー集です。アラートのトリアージ、ログ調査、再発防止のための調査自動化など、**夜間呼び出しの初動を AI に肩代わりさせる**実践パターンが整理されています。人手が薄い時間帯の一次対応を底上げしたいチームに刺さります。

### 3-3.〔公式〕Claude Code GitHub Actions で CI/CD を自動化
- 出典URL: https://code.claude.com/docs/en/github-actions
- 公式ドキュメント。`@claude` メンションや Issue 割り当てをトリガーに、**Issue→PR 作成、PRレビュー、テスト失敗の解析、changelog/リリースノート生成**などを GitHub Actions 上で自走させます。`CLAUDE.md` をリポジトリ直下に置けば、レビュー基準・コーディング規約・CIルールの「単一の真実の情報源」として全実行に効きます。コツは**プロンプトの具体性**（「レビューして」より「何を見るか」を明示）。

---

## 4. ベストプラクティス／グッドパターン

### 4-1. 私が実際に使っている DevOps 向け Claude Skill 8選（Pulumi）
- 出典URL: https://www.pulumi.com/blog/top-8-claude-skills-devops-2026/
- IaC ベンダー Pulumi による実践記事。kubernetes-specialist Skill が **`runAsNonRoot: true`・リソースの requests/limits・liveness/readiness probe・PodDisruptionBudget** など「本番で本当に必要な設定」を押さえてくれる、といった具体例が並びます。**「自社のTerraformモジュール構造・K8sマニフェスト規約・CI/CD段階・runbook を Skill に符号化する」**ことが本当の価値、という指摘が肝です。

---

## さらに試す価値（awesome シリーズ・リンク集）

- VoltAgent/awesome-claude-code-subagents ── 100+ の専門サブエージェント集（10カテゴリ）: https://github.com/VoltAgent/awesome-claude-code-subagents
- hesreallyhim/awesome-claude-code ── Skill・hook・スラッシュコマンド・オーケストレータの定番キュレーション: https://github.com/hesreallyhim/awesome-claude-code
- rohitg00/awesome-claude-code-toolkit ── エージェント/Skill/コマンド/プラグインを横断的に集めた大型ツールキット: https://github.com/rohitg00/awesome-claude-code-toolkit
- ahmedasmar/devops-claude-skills ── DevOps ワークフロー特化の Claude Skill マーケットプレイス: https://github.com/ahmedasmar/devops-claude-skills

> ⚠️ awesome 系・公開 Skill を導入する際は、上記 2-4 の通り**重大脆弱性・悪意あるペイロードの混入リスク**があります。出所・スター数・更新状況を確認し、可能なら中身を読んでから入れること。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
