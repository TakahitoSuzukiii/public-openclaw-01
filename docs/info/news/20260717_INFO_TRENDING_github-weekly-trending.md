# GitHub 週次トレンド（2026-07-17）

作成日: 2026-07-17 / STATUS: INFO / TOPIC: TRENDING

前回（2026-07-10）との差分をもとに、この1週間で伸びた・新しくランクインしたリポジトリをまとめます。対象プール 357 件、掲載基準はスター 10,000 以上（開発者向け）。優先トピック（go / rust / typescript / python / nextjs / claude / openclaw）は基準未満でも「注目枠」として拾います。

> 注: スター数は公式 GitHub REST API の取得値をそのまま記載。前週比（`+N`）はスナップショット差分です。用語の略語補足 —
> - **RAG**（Retrieval-Augmented Generation, 検索拡張生成）: 外部データを検索して LLM の回答に混ぜる手法。
> - **MCP**（Model Context Protocol）: AI エージェントに外部ツール/データを接続する共通プロトコル。
> - **CLI**（Command Line Interface）: コマンド操作のツール。
> - **Skill（スキル）**: Claude Code などのエージェントに手順・知識を追加する定義ファイル群。

---

## 今週の傾向（サマリ）

- **「エージェントに能力を足す Skill」系が引き続き主役。** スター増の上位はほぼ Skill／エージェント補助ツールで占められました（mattpocock/skills、obra/superpowers、ponytail、ui-ux-pro-max-skill など）。
- **コードを知識グラフ化する系が同時多発的に伸長。** Graphify、Understand-Anything、codebase-memory-mcp が揃ってランクイン。「巨大コードベースを LLM が少ないトークンで把握する」需要が明確化。
- **優先トピックの動き:** Python（AI/エージェント基盤）と Claude（Skill エコシステム）が量・伸びともに突出。TypeScript は Web スクレイピング/動画/デザイン系、Rust はエージェント CLI（openai/codex, cc-switch）が中心。OpenClaw はエージェント連携ツールでの言及が定着。

---

## 🔺 今週の急上昇（risers）

### mattpocock/skills（+11,094 → 175,575 ★｜Shell）
- **概要:** TypeScript 界隈で著名な Matt Pocock 氏が自身の `.agents` ディレクトリから公開している「実務エンジニア向け Skill」集。
- **なぜ注目か:** 今週の増加数トップ。著名開発者の実運用ノウハウがそのまま Skill 化されている点が支持を集めています。
- **主な特徴:** すぐ流用できる実務志向のスキル定義。
- **リンク:** <https://github.com/mattpocock/skills>

### obra/superpowers（+4,892 → 256,605 ★｜Shell）
- **概要:** エージェント用スキルフレームワーク＋ソフトウェア開発方法論。サブエージェント駆動開発（subagent-driven-development）を掲げます。
- **なぜ注目か:** 総スター 25 万超の大型リポジトリが今なお週 5,000 近く伸びており、Skill 方法論の中心的存在。
- **リンク:** <https://github.com/obra/superpowers>

### DietrichGebert/ponytail（+5,094 → 85,187 ★｜JavaScript）★優先: claude
- **概要:** 「部屋で一番怠惰なシニア開発者のように考えさせる」= YAGNI（You Aren't Gonna Need It, 必要になるまで作るな）を徹底させる Claude Code 向けスキル。
- **なぜ注目か:** 6月中旬公開の新顔ながら急伸。「一番良いコードは書かなかったコード」という思想がウケています。
- **リンク:** <https://github.com/DietrichGebert/ponytail>

### firecrawl/firecrawl（+3,511 → 152,379 ★｜TypeScript）★優先: typescript
- **概要:** Web を大規模に検索・スクレイプし、LLM 向けに Markdown 化する API。
- **なぜ注目か:** RAG やエージェントの「Web を読む」土台として定番化。安定した伸びが続いています。
- **主な特徴:** HTML→Markdown 変換、クローラ、AI 検索を統合。
- **リンク:** <https://github.com/firecrawl/firecrawl>

### そのほかの主な急上昇
- **Shubhamsaboo/awesome-llm-apps**（+6,092 → 123,562 ★｜Python）: 実際に動かせる 100+ の AI エージェント/RAG アプリ集。
- **NousResearch/hermes-agent**（+3,704 → 216,418 ★｜Python）★優先: python/claude/openclaw: 「あなたと共に成長するエージェント」。Anthropic/OpenAI/OpenClaw など幅広く対応。
- **Panniantong/Agent-Reach**（+3,042 → 57,470 ★｜Python）★優先: python/claude: Twitter/Reddit/YouTube/GitHub 等を API 料金ゼロで読ませる CLI。
- **nextlevelbuilder/ui-ux-pro-max-skill**（+3,011 → 106,992 ★）★優先: python/claude: UI/UX デザイン知能を与える Skill。
- **multica-ai/andrej-karpathy-skills**（+3,011 → 193,577 ★）★優先: claude: Karpathy 氏の LLM コーディング落とし穴観察を反映した 1 枚の `CLAUDE.md`。
- **DeusData/codebase-memory-mcp**（+2,759 → 32,402 ★｜C）★優先: claude: コードを知識グラフ化する高速 MCP サーバ（158 言語・サブ ms クエリ）。

---

## 🆕 新規ランクイン（newcomers）

### OpenCut-app/OpenCut（74,746 ★｜TypeScript）★優先: typescript
- **概要:** オープンソースの CapCut 代替となる動画エディタ。
- **なぜ注目か:** 2025年6月作成の比較的新しいプロジェクトが一気にプール入り。ブラウザ動画編集の OSS 需要を反映。
- **リンク:** <https://github.com/OpenCut-app/OpenCut>

### odysseus-dev/odysseus（83,109 ★｜Python）★優先: python
- **概要:** セルフホスト型の AI ワークスペース。
- **なぜ注目か:** 2026年5月末作成の新顔で 8 万★超に到達。手元完結の AI 環境ニーズを示します。
- **リンク:** <https://github.com/odysseus-dev/odysseus>

### rasbt/LLMs-from-scratch（99,263 ★｜Jupyter Notebook）★優先: python
- **概要:** PyTorch で ChatGPT 相当の LLM をゼロから段階的に実装する教材。
- **なぜ注目か:** 学習リソースとして定番化。基礎から仕組みを理解したい層に刺さっています。
- **リンク:** <https://github.com/rasbt/LLMs-from-scratch>

### anthropics/claude-cookbooks（49,086 ★｜Jupyter Notebook）★優先: claude
- **概要:** Claude を効果的に使うためのレシピ/ノートブック集（Anthropic 公式）。
- **なぜ注目か:** 公式のベストプラクティス集。エージェント開発の一次情報源。
- **リンク:** <https://github.com/anthropics/claude-cookbooks>

### iOfficeAI/OfficeCLI（18,781 ★｜C#）★優先: claude/openclaw
- **概要:** AI エージェントが Word/Excel/PowerPoint を読み書き・自動化するための Office スイート CLI。Office 本体不要の単一バイナリ。
- **なぜ注目か:** エージェントに「オフィス文書操作」を与える実用ツール。OpenClaw/Claude Code 連携を明示。
- **リンク:** <https://github.com/iOfficeAI/OfficeCLI>

### そのほかの新規（大型・一般向け）
- **codecrafters-io/build-your-own-x**（527,218 ★｜Markdown）: 好きな技術をゼロから再実装して学ぶ定番リスト。
- **DigitalPlatDev/FreeDomain**（186,476 ★）: 無料ドメイン提供プラットフォーム。
- **swisskyrepo/PayloadsAllTheThings**（79,237 ★｜Python）: Web セキュリティ/ペンテストのペイロード集。
- **v2ray/v2ray-core**（46,923 ★｜Go）★優先: go: プロキシ構築プラットフォーム。

---

## ⭐ 優先トピック注目枠（基準未満を含む）

優先トピックのうち、上の急上昇/新規で拾いきれなかったものを補足します。

### Rust
- **openai/codex**（99,140 ★）: ターミナルで動く軽量コーディングエージェント。Rust 製エージェント CLI の代表格。<https://github.com/openai/codex>
- **farion1231/cc-switch**（118,331 ★）★優先: rust/typescript/claude/openclaw: Claude Code / Codex / OpenClaw / Gemini CLI などを一括管理するクロスプラットフォーム・デスクトップ補助（Tauri）。<https://github.com/farion1231/cc-switch>

### Next.js
- **KuekHaoYang/KVideo**（3,842 ★｜TypeScript）★優先: typescript/nextjs: Next.js 16 製の動画アグリゲーション再生プラットフォーム。「Liquid Glass」デザインが特徴。基準（1万）未満だが nextjs 該当のため注目枠として掲載。<https://github.com/KuekHaoYang/KVideo>

### TypeScript（デザイン/動画/知識グラフ）
- **nexu-io/open-design**（79,296 ★）★優先: typescript/claude: OSS の「Claude Design 代替」。ローカルファーストでプロトタイプ/LP/スライド/画像/動画を生成。<https://github.com/nexu-io/open-design>
- **Egonex-AI/Understand-Anything**（74,896 ★）★優先: typescript/claude: 任意コードを対話可能な知識グラフに変換。<https://github.com/Egonex-AI/Understand-Anything>

### OpenClaw エコシステム
- **kepano/obsidian-skills**（42,390 ★）★優先: claude/openclaw: Obsidian CLI と開放フォーマット（Markdown/Bases/JSON Canvas）をエージェントに教える Skill。<https://github.com/kepano/obsidian-skills>
- （上記の Graphify / hermes-agent / cc-switch / OfficeCLI も OpenClaw 対応を明示）

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
