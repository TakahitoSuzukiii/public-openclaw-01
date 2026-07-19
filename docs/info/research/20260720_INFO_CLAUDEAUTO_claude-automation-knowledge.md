作成日: 2026-07-20 / STATUS: INFO / TOPIC: CLAUDEAUTO

# 週次まとめ：Claude / Claude Code による業務自動化ナレッジ（2026-07-20 号）

> 注: 各記事の主張・数値・手順は出典元のものです。製品仕様やコマンドは変わり得るため、導入時は必ず一次情報で再確認してください。本記事は要約・考察であり全文転載ではありません。専門用語は綴りと意味を併記します。

## 今週のテーマ・見どころ

先週（7/13 号）は Excel / PowerPoint を「出力する」File Skills を主役にしました。今週はそこから一歩進めて、**エージェントを一度きりの作業で終わらせず『仕組み（再現可能な自動化）』に落とし込む**ことに焦点を当てます。具体的には次の3系統です。

- **ブラウザ操作の自動化** ── Playwright（プレイライト＝Microsoft 製のブラウザ自動操作フレームワーク）を Claude Code から呼び出し、テスト・仕様書作成・情報収集を任せる。
- **拡張の三本柱（hooks / subagents / plugins）** ── 「毎回同じことを必ずやらせる」仕組み化のための土台。
- **セキュリティ運用の自動化（SOC 適用）** ── 脆弱性トリアージやアラート仕分けを AI に肩代わりさせる最新動向。

最後に Excel/PPT の M365（Microsoft 365）統合の続報と、awesome 系のリンク集を添えます。

---

## 1. ブラウザ操作の自動化 ── Playwright × Claude Code（今週の主役）

RPA（Robotic Process Automation＝定型的な画面操作の自動化）的な作業を、スクリプトを手書きせず自然言語で任せられるのが今の潮流です。IT インフラ現場でも「管理コンソールの定期確認」「証跡スクショの取得」「Web フォーム入力」など、地味だが数の多い作業に効きます。

### 1-1. DevContainer で完結させる Playwright MCP 構築手順
- 出典URL: https://zenn.dev/secondselection/articles/claude_playwright
- Claude Code ＋ Playwright MCP（Model Context Protocol＝AI とツールをつなぐ標準規格）を **DevContainer（VS Code の開発コンテナ）内に閉じ込めて**構築する手順。ローカル環境を汚さず、チームで同じブラウザ自動化環境を再現できるのが利点です。まず「環境の再現性」を担保したい現場向けの入口として実用的。

### 1-2. Playwright MCP でブラウザを操作する 5 ステップ（最小実装）
- 出典URL: https://note.com/cc_works_cl/n/nd1a9d548a4da
- Claude Code から Playwright MCP を呼び出す最小構成を 5 ステップで解説。AI は Playwright の API を直接書かず、ツール名とパラメータを渡すだけでブラウザを操作します。「まず 30 分で動かす」ための足がかりとして。

### 1-3. Agent Skills × Playwright MCP で画面仕様書を自動作成
- 出典URL: https://iret.media/191237
- Claude Code の Agent Skills とブラウザ操作を組み合わせ、**AI が自律的にサイトを巡回して画面仕様書を書き上げる**全工程の実録。既存 Web システムのドキュメント化という「後回しにされがちなトイル（toil＝価値を生まない反復作業）」を丸ごと委譲できる好例です。

### 1-4. Playwright と browser-use の使い分け（トークン効率の実測）
- 出典URL: https://fyve.co.jp/claude-code/articles/browser-use-vs-playwright-guide
- 補足（比較検証）: https://zenn.dev/shinyaa31/articles/dd315ea4868eb1
- Playwright MCP と、軽量な agent-browser／browser-use 系の比較。記事によると同一タスク 6 件で Playwright MCP が約 7,800 トークン消費に対し軽量系は約 1,400 トークンに収まった、という実測値も。**精密な検証は Playwright、手数の多い巡回は軽量系**という使い分けの指針になります（数値は出典元の測定値。導入時は自環境で再確認を）。

---

## 2. 拡張の三本柱 ── hooks / subagents / plugins で「仕組み化」する

単発のプロンプトは属人化しがちです。Claude Code の拡張機構を使うと、自動化を**チームで共有・再現できる資産**に変えられます。

### 2-1.〔公式〕Hooks で「必ず実行される」動作を仕込む
- 出典URL: https://code.claude.com/docs/en/hooks-guide （公式ドキュメント）
- hooks（フック）は、エージェントのライフサイクル上の特定タイミングで**確定的に**処理を差し込む仕組み。`PreToolUse`（ツール実行前の安全チェックポイント）、`UserPromptSubmit`（プロンプト送信時）、`PermissionRequest`（権限要求時の自動承認/拒否）などがあり、「LLM の気分に依存せず、決められた検査・整形を毎回走らせる」用途に向きます。インフラ運用で「破壊的コマンドの前に必ず確認を挟む」といったガードレールに。

### 2-2. hooks / subagents / skills / plugins の実践的な全体像
- 出典URL: https://ofox.ai/blog/claude-code-hooks-subagents-skills-complete-guide-2026/
- 参考（実践概観）: https://medium.com/@mishra.shashank35/claude-code-skills-subagents-hooks-and-plugins-a-practical-overview-572de7cedb20
- 4 つの拡張要素の関係を整理。**subagents（サブエージェント）**は独立したコンテキストで作業し、詳細ログを隔離して要約だけを親に返すため、大量のログ解析やコード探索で本体の文脈を汚しません。**plugin（プラグイン）**は skills/subagents/commands/hooks/MCP をひとまとめにした配布単位で、`/plugin` 一発でチーム全員に同じ構成を配れます。おすすめの順序は「まず skills → 確定的な強制が要るなら hooks → 並列・文脈隔離が要るなら subagents」。

### 2-3. 25 機能を例つきで俯瞰するガイド
- 出典URL: https://www.marktechpost.com/2026/06/14/claude-code-guide-2026-25-features-with-examples-demo/
- hooks が発火する 25 のライフサイクルポイントや、判断を要する場面で使う「prompt ベース/agent ベースの hook」など、機能を例つきで一望できるカタログ。自動化のネタ探しに。

---

## 3. セキュリティ運用の自動化 ── SOC でのトリアージと公開ベータ

先週は「脆弱性を見つける」Claude Code Security を扱いました。今週はその**運用（SOC＝Security Operations Center＝セキュリティ監視の司令塔）への組み込み**と最新アップデートに寄せます。

### 3-1.〔公式〕Claude Code Security が公開ベータへ（防御側への提供）
- 出典URL: https://www.anthropic.com/news/claude-code-security （公式）
- 参考（報道）: https://venturebeat.com/security/anthropic-claude-code-security-reasoning-vulnerability-hunting
- 2026 年 2 月のプレビューを経て、Claude Security が **Claude Enterprise 向け公開ベータ**に移行。追加された 5 機能は「定期スキャン（継続監視）」「ディレクトリ単位のターゲティング（大規模コードベース向け）」「棄却理由を残す構造化トリアージ追跡」「CSV/Markdown エクスポート（コンプライアンス報告向け）」「Slack/Jira への Webhook 連携」。**棄却理由＋タイムスタンプが監査証跡になり、SOC 2 / ISO 27001 の要件に沿う**点が運用上のポイントです。

### 3-2. SOC で Claude を「force multiplier」にする
- 出典URL: https://dcloud9.com/blog/claude-cybersecurity-2026.html
- 慎重に配備すれば、Claude は Tier-1 アナリストより文脈のあるアラート・トリアージ（優先度づけ）を行い、多数の脅威インテリジェンスフィードを数分で統合できる、という現場観点の解説。Splunk / Sentinel / CrowdStrike 等に組み込む「アラート仕分け・脅威インテリジェンス統合・脆弱性優先度づけ・フィッシング分類」エージェントの構図を紹介します。人手の一次対応を減らす設計の参考に。

### 3-3. 500 件超のゼロデイと「誰が直すのか」問題
- 出典URL: https://futurumgroup.com/insights/claude-found-500-zero-days-who-patches-them-before-attackers-arrive/
- Frontier Red Team が報告した「本番 OSS で 500 件超の高深刻度脆弱性を発見・検証」の含意を、**発見の次＝修正の運用（誰が・いつ・どう直すか）**の観点で論じた記事。発見能力が上がるほど、修正フローとトリアージ体制の整備が本丸になる、という示唆です。

---

## 4. Excel / PowerPoint 自動化：M365 統合の続報

先週の File Skills に対し、今週は **アドイン（Excel/PowerPoint 内で動く Claude）と M365 統合**側の続報を短く。

### 4-1. Excel → PowerPoint の「文脈を引き継いだ」変換
- 出典URL: https://uravation.com/media/claude-excel-powerpoint-addin-microsoft365-2026/
- 参考（1発変換の解説）: https://shizuoka-marketing.co.jp/ai/claude-cowork-excelpowerpoint/
- 2026 年の M365 統合更新で、Excel で分析した内容を**文脈ごと PowerPoint に引き継ぐ**運用が可能に。経理での「部門別集計→予算対比・前年同月比→取締役会用スライド自動生成」といった定型フローが一気通貫になります。数値・仕様は出典元の記載なので、社内では小さく検証してから展開を。

---

## さらに試す価値：awesome / workflow レシピ集（リンク集）

- **ithiria894/awesome-claude-code-workflows** ── hooks・MCP・skills・agents・CLAUDE.md を束ねた「実タスク自動化レシピ」集: https://github.com/ithiria894/awesome-claude-code-workflows
- **ithiria894/awesome-claude-code-hooks** ── イベント駆動自動化のための hooks 専門キュレーション: https://github.com/ithiria894/awesome-claude-code-hooks
- **hesreallyhim/awesome-claude-code** ── skills/agents/status lines/plugins の定番総合カタログ（約 30k stars）: https://github.com/hesreallyhim/awesome-claude-code
- **rohitg00/awesome-claude-code-toolkit** ── 135 agents / 35 skills / 176+ plugins / 20 hooks 等の大型ツールキット: https://github.com/rohitg00/awesome-claude-code-toolkit
- **disler/claude-code-hooks-mastery** ── hooks を体系的に学ぶ実装リポジトリ: https://github.com/disler/claude-code-hooks-mastery
- **8 Claude Code Workflows With Real Use Cases (2026)** ── 実務由来の 8 ワークフローを、正直なトレードオフ付きで紹介: https://www.ayautomate.com/blog/best-claude-code-workflows

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
