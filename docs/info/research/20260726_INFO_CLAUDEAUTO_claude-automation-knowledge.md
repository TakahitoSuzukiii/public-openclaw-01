作成日: 2026-07-26 / STATUS: INFO / TOPIC: CLAUDEAUTO

# 週次まとめ：Claude / Claude Code による業務自動化ナレッジ（2026-07-26 号）

> 注: 各記事の主張・数値・手順は出典元のものです。製品仕様やコマンドは変わり得るため、導入時は必ず一次情報で再確認してください。本記事は要約・考察であり全文転載ではありません。専門用語は綴りと意味を併記します。

## 今週のテーマ・見どころ

先週（7/20 号）は「エージェントを単発で終わらせず、hooks / subagents / plugins で**仕組み化**する」ことを軸にしました。今週はその一段先、**『人の判断そのものを Skill として定義し、運用フローに埋め込む』実践事例**を主役に据えます。焦点は次の3系統です。

- **SRE 現場での「判断の仕組み化」** ── 監視・チケット運用の「暗黙知」を Skill にコード化し、レビュー可能な資産に変える。
- **CI/CD への組み込み** ── GitHub Actions 上で Claude Code を「人のいない安全網（自動レビュー/自動保守）」として常駐させる。
- **セキュリティ運用の自動化（検出→トリアージ→修復のループ）** ── Claude Security 公開ベータの新機能と SOC 適用。

加えて、優先トピックである **Excel/PowerPoint レポート自動生成の続報**、そして**定期実行基盤（Managed Agents / Cowork）**、末尾に awesome 系リンク集を添えます。

用語補足: SRE=Site Reliability Engineering＝サイト信頼性工学（運用を工学的に扱う職種）、CI/CD=Continuous Integration / Continuous Delivery＝継続的統合／継続的デリバリー、SOC=Security Operations Center＝セキュリティ監視・対応チーム、MCP=Model Context Protocol＝AI と外部ツールをつなぐ標準規格。

---

## 1. インフラ運用・SRE ── 「人による判断」を Skill で定義する（今週の主役）

数の多い運用作業ほど「何をもって異常とするか」が担当者の経験に依存しがちです。今週の目玉は、その**判断基準そのものを自然言語で書いて Skill 化し、レビュー・改善の対象にした**現場記録です。

### 1-1.〔実践〕SRE が「人による判断」を Claude Code で定義にした話
- 出典URL: https://creators-note.chatwork.com/entry/2026/06/30/105124
- SRE チームが Claude Code の Skill を運用に組み込んだ実録。ポイントは3つ。①**チケット運用の標準化** ── Skill が起票時に「背景・実施内容・完了条件」を対話形式で収集し、別エージェントがスコープの具体性を点検。経験頼みだった判断が文書化され手戻りが減少。②**ダッシュボード監視の一元化** ── メトリクス取得はスクリプトが並列実行し、Claude は「ステータス判定と文章化」に専念。担当者によらず同じ尺度で評価できる。③**判断基準のコード化** ── 暗黙知だった判定ロジックを Skill として PR（プルリクエスト）対象にし、レビュー・改善サイクルに乗せた。「判定基準を自然言語で書ける」ことこそが Claude を使う利点、という結論が示唆的です。

### 1-2. 部署別・業種別に見る Claude Code 活用の類型
- 出典URL: https://uravation.com/media/claude-code-use-cases-10-by-department-industry-2026/
- 参考（実装パターン別 ROI 解説）: https://uravation.com/media/claude-code-implementation-cases-10-japan-2026/
- 日本企業の導入を「部署別・業種別」「実装パターン別」に整理したまとめ。レガシーコードのリファクタリング自動化、テストコード一括生成、社内ドキュメントの自動更新などが代表例として挙がります。自分の現場に近い型を探す「地図」として有用（数値・ROI は出典元の主張のため、導入時は自環境で検証を）。

---

## 2. CI/CD への組み込み ── GitHub Actions で「人のいない安全網」を作る

Claude Code をローカルの開発補助だけで使うのは半分もったいない、というのが今の潮流です。**GitHub Actions に載せて PR レビュー・課題トリアージ・定期保守を自動化**する構成が定着してきました。

### 2-1.〔公式アクション〕`/install-github-app` で PR 自動レビューを即設置
- 出典URL: https://systemprompt.io/guides/claude-code-github-actions
- 参考（総合ガイド）: https://aiforanything.io/blog/claude-code-github-actions-cicd-integration-guide-2026
- `anthropics/claude-code-action` を使い、PR レビュー・issue トリアージ・定期メンテナンスを人手なしで回す構成の手引き。Claude Code 内で `/install-github-app` を実行すれば、動く GitHub Actions ワークフローが数秒で生成されます。**推奨構成は「ローカル Claude Code で開発 ＋ GitHub Actions で自動安全網」の二段構え**。main に届く前に問題を捕まえます。

### 2-2. CI で失敗しないための実務ノウハウ（プロンプト品質・秘密情報・耐障害性）
- 出典URL: https://opentools.ai/resources/claude-code-cicd-github-actions
- CI で回す際の勘所を整理。①**プロンプト品質**＝CI では対話できないので「何を分析し、どんな出力形式で、どの範囲か」を明示し、`CLAUDE.md`（毎回読み込まれる恒久設定）にチーム規約を書いておく。②**秘密情報**＝API キーは GitHub Secrets に置き、ハードコード禁止。権限は必要最小限に絞る。③**耐障害性**＝レート制限やタイムアウトは必ず起きるので `continue-on-error` と `timeout` で暴走コストを防ぐ。地味だが「本番で事故らない」ための必読ポイント。

---

## 3. Excel / PowerPoint レポート自動生成（優先トピック・続報）

インフラ運用でも数の多い「月次レポート・報告資料づくり」。File Skills 系の続報として、**M365（Microsoft 365）統合と Skill 対応の進展**を押さえます。

### 3-1. Skill で Excel → PowerPoint → PDF を一気通貫
- 出典URL: https://zenn.dev/ttks/articles/963d86a67a2083
- Claude API の Skills で Excel・PowerPoint・PDF を自動生成する実践。典型ワークフローは「Excel の生データを読む → 前月比を計算 → PowerPoint 化 → PDF 保存」を**単一 Skill で完結**させるもの。Anthropic 公式のプリビルト Skill（Excel/PowerPoint/Word/PDF）があり、Progressive Disclosure（必要な指示だけ段階的に読み込む仕組み）でトークン消費を抑えます。

### 3-2. Claude for Excel / PowerPoint に3つの改善（コンテキスト同期・Skill 対応）
- 出典URL: https://forest.watch.impress.co.jp/docs/news/2093856.html
- 参考（M365 統合・Copilot 比較）: https://uravation.com/media/claude-excel-powerpoint-addin-microsoft365-2026/
- 2026年、Claude for Excel と Claude in PowerPoint が Skill 対応・コンテキスト同期などの改善を受けたニュース。**企業ごとの定型業務を Skill として定義し、チームに配布**できるのが実務上の肝。「資料作成の属人化」を解消したい部署に効きます。

---

## 4. ゼロデイ／セキュリティ運用の自動化（優先トピック）

脆弱性対応は「見つける」だけでなく「仕分けて直す」まで含めて数が多い。今週は **Claude Security の検出→トリアージ→修復の一貫ループ**と SOC 適用の動向です。

### 4-1.〔公式〕Claude for Cybersecurity ── 検出・トリアージ・修復を1つのループに
- 出典URL: https://claude.com/solutions/cybersecurity
- 脅威モデリング → 発見 → 検証 → トリアージ → パッチを「一続きのループ」として扱い、各段階でコンテキストを保持するのが設計思想。Code Review skill による PR の自動セキュリティレビュー（ロジック誤り・脆弱性・デグレの検出）、Threat Model skill による「コード由来の脅威モデル → ATT&CK マッピング」まで対応。トリアージ／パッチング／検出のスキルは GitHub で参考実装が公開され、組織ごとにカスタマイズ可能です。

### 4-2. Claude Security 公開ベータの5つの新機能（運用・監査向け）
- 出典URL: https://wowhow.cloud/blogs/claude-security-public-beta-ai-vulnerability-scanning-enterprise-guide-2026
- 公開ベータで加わった5機能：①**定期スキャン**（継続監視）、②**ディレクトリ単位のターゲティング**（大規模コードベース向け）、③**構造化トリアージ**（却下理由を記録）、④**CSV/Markdown エクスポート**（コンプライアンス報告）、⑤**Slack / Jira 連携（webhook）**。却下ごとに理由と時刻を残すため、SOC 2 / ISO 27001 などの監査証跡になります。「見つけて終わり」でなく**運用に乗せる**ための機能群。

### 4-3. SOC ワークフローへの適用パターン
- 出典URL: https://torq.io/blog/claude-mythos-soc/
- 参考（SOC 向け観点整理）: https://www.blockchain-council.org/claude-ai/claude-mythos-for-cybersecurity-threat-intelligence-incident-triage-soc-reporting/
- アラートトリアージ・脅威インテリジェンス統合・脆弱性の優先順位付け・フィッシング分類などを、Splunk / Sentinel / CrowdStrike 等に配線するプリビルトエージェントの考え方。**人の一次仕分けを肩代わりさせ、アナリストは判断が要る案件に集中**という分業モデルが要点です（製品連携は自環境の要件・ライセンスを要確認）。

---

## 5. 定期実行の基盤 ── Managed Agents / Cowork で「回し続ける」

単発の便利さを「勝手に回り続ける仕組み」に変える土台として、スケジュール実行と耐障害運用の動向も押さえます。

### 5-1. Claude Code Managed Agents ── スケジュール実行と CLI 統合
- 出典URL: https://qiita.com/kai_kou/items/337eb235da9b9a081836
- 2026年6月にパブリックベータ入りした Managed Agents の入門。スケジュール実行・CLI 統合・Vault 管理の環境変数を組み込んだ自動化基盤で、「毎時トレンド監視」「定期コンテンツ生成」のような cron 的運用を公式機構で実現できます（本ニュース自体、まさにこの種の定期タスクの成果物です）。

### 5-2. 自動化を止めない運用術（fallbackModel / --safe-mode / /cd）
- 出典URL: https://qiita.com/kai_kou/items/a54258770d3a5e9d9ee0
- 2026年6月アップデートで追加された、自動化パイプラインを止めず素早く切り分けるための機能群。`fallbackModel`（モデル障害時の代替）、`--safe-mode`、`/cd` などを使い、無人運用でも「詰まったら安全に縮退する」設計にする考え方。定期タスクの信頼性を上げたい人向け。

### 5-3. Claude Cowork ── デスクトップの定型オフィス作業を自律実行
- 出典URL: https://9to5mac.com/2026/04/09/anthropic-scales-up-with-enterprise-features-for-claude-cowork-and-managed-agents/
- 参考（企業向けセキュリティ観点）: https://www.paloaltonetworks.com/blog/2026/07/what-it-takes-to-secure-claude-cowork-across-the-ai-enterprise/
- ファイル整理・資料作成・調査の統合など、時間を食う定型作業をデスクトップ上で自律実行する Cowork。2026年4月からは RBAC（役割ベースのアクセス制御）、グループ支出上限、利用分析、OpenTelemetry 拡張などエンタープライズ機能が追加。「個人の便利」から「組織で統制して使う」段階に入りつつあります（導入時はデータ持ち出し・権限設計の検討が前提）。

---

## さらに試す価値 ── awesome シリーズ／リンク集

- **awesome-claude-code**（本家・手選りコレクション）: https://github.com/hesreallyhim/awesome-claude-code
- **awesome-claude-code-subagents**（100+ の専門サブエージェント）: https://github.com/VoltAgent/awesome-claude-code-subagents
- **awesome-claude-code-toolkit**（agents/skills/commands/plugins/hooks を網羅）: https://github.com/rohitg00/awesome-claude-code-toolkit
- **awesome-claude-code-and-skills**（Skill 集）: https://github.com/GetBindu/awesome-claude-code-and-skills
- **GitHub Topics: claude-code-skills**（新着 Skill の定点観測）: https://github.com/topics/claude-code-skills
- **Claude Platform Docs ─ Agent Skills クイックスタート（公式）**: https://platform.claude.com/docs/ja/agents-and-tools/agent-skills/quickstart

> ヒント: まず本家 awesome-claude-code で全体像を掴み、用途が定まったら subagents / toolkit 系で具体的な実装例をコピーして自環境に合わせて調整するのが効率的です。公開スキルは中身を必ず読んでから使うこと（権限・外部通信の有無を確認）。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
