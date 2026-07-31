# GitHub 週次トレンド（2026-07-31）

作成日: 2026-07-31 / STATUS: INFO / TOPIC: TRENDING

> 対象期間: 2026-07-24 → 2026-07-31（前週比）
> データ元: GitHub 公式 REST API（`search/repositories`）のみ使用。Web スクレイピングは不使用。
> 掲載基準: スター 10,000 以上・開発者向け。ただし優先トピック（go / rust / typescript / python / nextjs / claude / openclaw ほか）は「注目枠」として基準未満でも取り上げます。
> 集計プール: 452 リポジトリ / 新規ランクイン 22 / スター増リポジトリ 415 / 優先トピックヒット 386。

---

## 今週の概観（要点）

- **今週の主役は「AI エージェント向けスキル / ハーネス」系**。前週比のスター増（riser）上位は、ほぼすべて Claude Code・Codex・Cursor などの「AIコーディングエージェントを強化するツール」で占められました。
- とりわけ **`odysseus-dev/odysseus`（+22,526）** が単独で突出。セルフホスト型 AI ワークスペースとして一気に伸びています。
- **トークン削減（token optimization）** が明確なテーマに。`OmniRoute`・`rtk`・`CLIProxyAPI` など「LLM のコストを下げるゲートウェイ/プロキシ」が複数ランクイン。
- **コードをナレッジグラフ化する**系（`graphify` / `codegraph` / `Understand-Anything`）も継続して強い。「LLM に大規模コードを理解させる」需要が高止まりしています。
- 優先トピックでは **openclaw** タグを持つ大型リポ（`hermes-agent`・`airi`・`graphify`・`cc-switch`）が軒並み伸長。

※用語補足: **ハーネス（harness）** = AI エージェントを動かすための土台・実行環境のこと。**スキル（skill）** = エージェントに特定作業をやらせるための追加手順書/プラグイン。**ゲートウェイ（gateway）** = 複数の LLM API を1つの窓口にまとめる中継サーバ。

---

## 🔥 今週の急上昇（Risers・前週比スター増 上位）

### 1. odysseus-dev/odysseus （Python） — ★84,404（前週比 +22,526）
- **概要**: セルフホスト型の AI ワークスペース（"Self-hosted AI workspace"）。自分のサーバ上で AI 作業環境を丸ごと持てるツール。
- **なぜ注目か**: 今週の増加数がダントツ1位。5月末公開の新興ながら3週足らずで8万スター超へ。データを自分の手元に置きたい層に強く刺さっています。
- **主な言語**: Python
- **リンク**: <https://github.com/odysseus-dev/odysseus>

### 2. mattpocock/skills （Shell） — ★197,707（+11,152）
- **概要**: TypeScript 界隈で著名な Matt Pocock 氏が自分の `.agents` ディレクトリから公開した「実務エンジニア向けスキル集」。
- **なぜ注目か**: 著名開発者の実運用ノウハウがそのまま出た形。スキル系リポの中でも母数が大きく、増加も大きい。
- **主な言語**: Shell
- **リンク**: <https://github.com/mattpocock/skills>

### 3. diegosouzapw/OmniRoute （TypeScript / claude）— ★36,005（+7,325）
- **概要**: MIT ライセンスの無料 AI ゲートウェイ。1エンドポイントで 290+ プロバイダ・500+ モデル（Claude / GPT / Gemini / DeepSeek ほか）へ接続。クォータ対応の自動フォールバック、トークン圧縮で 15〜95% 削減を謳う。
- **なぜ注目か**: 「複数モデルを1つの窓口で・安く」という今週の主要テーマ（トークン削減＋マルチプロバイダ）の代表格。
- **主な特徴**: 自動フォールバック / MCP・A2A 対応 / Claude Code・Codex・Cursor 連携。
- **主な言語**: TypeScript
- **リンク**: <https://github.com/diegosouzapw/OmniRoute>

### 4. stablyai/orca （TypeScript / claude）— ★34,578（+6,301）
- **概要**: 並列に走る複数エージェントの「艦隊」を扱う ADE（Agent Development Environment）。デスクトップ・モバイル・VPS で動作。
- **なぜ注目か**: 「1人で何体もの AI エージェントを同時に走らせる」並列エージェント運用の需要増を反映。YC 支援。
- **主な言語**: TypeScript
- **リンク**: <https://github.com/stablyai/orca>

### 5. Graphify-Labs/graphify （Python / claude / openclaw）— ★99,686（+4,502）
- **概要**: コードベース・ドキュメント・SQLスキーマ・PDF などを、クエリ可能なナレッジグラフに変換。ローカルの決定論的 AST 解析（構文木解析）で、ベクトルストア不要。
- **なぜ注目か**: openclaw タグ持ちの大型リポ。「LLM に大規模コードを正確に理解させる」用途で定番化しつつある。
- **主な言語**: Python
- **リンク**: <https://github.com/Graphify-Labs/graphify>

### 6. earendil-works/pi （TypeScript）— ★81,451（+4,432）
- **概要**: AI エージェント用ツールキット。統合 LLM API・エージェントループ・TUI・コーディングエージェント CLI を一式提供。
- **なぜ注目か**: 「エージェントを作る土台」を丸ごと提供する系として着実に伸長。
- **主な言語**: TypeScript
- **リンク**: <https://github.com/earendil-works/pi>

### 7. koala73/worldmonitor （TypeScript）— ★77,399（+4,249）
- **概要**: リアルタイムの世界情勢ダッシュボード。AI によるニュース集約・地政学モニタリングを統合。
- **なぜ注目か**: エージェント＋MCP を使った「情報の集約・可視化」アプリの人気例。
- **主な言語**: TypeScript
- **リンク**: <https://github.com/koala73/worldmonitor>

### 8. DietrichGebert/ponytail （JavaScript / claude）— ★92,991（+4,049）
- **概要**: AI エージェントに「部屋で一番ぐうたらなシニア開発者」のように考えさせるスキル。「最良のコードは書かずに済ませたコード」（YAGNI 思想）。
- **なぜ注目か**: 過剰実装を抑える系のプロンプト/スキルとして話題継続。
- **主な言語**: JavaScript
- **リンク**: <https://github.com/DietrichGebert/ponytail>

### 9. obra/superpowers （Shell）— ★264,429（+3,877）
- **概要**: エージェント用スキルフレームワーク＋ソフト開発方法論。サブエージェント駆動開発（subagent-driven development）を掲げる。
- **なぜ注目か**: 母数25万超の超大型。安定して伸び続けるスキル系の代表。
- **リンク**: <https://github.com/obra/superpowers>

### 10. affaan-m/ECC （JavaScript / claude）— ★236,614（+3,739）
- **概要**: エージェントハーネスの性能最適化システム。スキル・記憶・セキュリティ・リサーチ先行の開発手法を統合。
- **リンク**: <https://github.com/affaan-m/ECC>

その他の急上昇: `pbakaus/impeccable`（+3,704・デザイン言語）/ `NousResearch/hermes-agent`（+3,403・openclaw）/ `firecrawl/firecrawl`（+3,142・Web取得API）/ `moeru-ai/airi`（+3,132・openclaw）/ `Panniantong/Agent-Reach`（+2,775・各種SNSをCLIで横断取得）。

---

## ⭐ 優先トピック 注目枠

### go
- **router-for-me/CLIProxyAPI** — ★45,817（今週新規プールイン / go・claude）。Antigravity・Codex・Claude Code・Grok Build を OpenAI/Gemini/Claude 互換 API として包み、無料枠モデルを API 経由で使えるようにするプロキシ。<https://github.com/router-for-me/CLIProxyAPI>

### rust
- **helix-editor/helix** — ★45,678（新規プールイン）。モーダル操作のモダンなテキストエディタ（Vim/Kakoune 系譜、Rust 製）。<https://github.com/helix-editor/helix>
- **rtk-ai/rtk** — ★74,176（rust・claude）。開発コマンドの LLM トークン消費を 60〜90% 削減する CLI プロキシ。依存ゼロの単一 Rust バイナリ。<https://github.com/rtk-ai/rtk>
- **farion1231/cc-switch** — ★122,903（rust・typescript・claude・openclaw）。Claude Code / Codex / OpenClaw / Hermes Agent などのプロバイダを切替える Tauri 製デスクトップアシスタント。<https://github.com/farion1231/cc-switch>

### typescript
- **anomalyco/opencode** — ★191,638。オープンソースのコーディングエージェント本体。<https://github.com/anomalyco/opencode>
- **n8n-io/n8n** — ★198,871。ネイティブ AI 機能付きのワークフロー自動化基盤（400+ 連携・セルフホスト可）。<https://github.com/n8n-io/n8n>

### python
- **Shubhamsaboo/awesome-llm-apps** — ★129,345。100+ の AI エージェント/スキル/RAG アプリを集めた無料 OSS 集。<https://github.com/Shubhamsaboo/awesome-llm-apps>
- **github/spec-kit** — ★124,789。仕様駆動開発（Spec-Driven Development）のツールキット。<https://github.com/github/spec-kit>
- **QwenPaw / agentscope-ai**（新規・★31,454）、**HKUDS/Vibe-Trading**（新規・★28,966・AI トレーディングエージェント）も今週プールイン。

### nextjs
- **shuding/nextra** — ★13,882（新規プールイン / typescript・nextjs）。Next.js 上で作るシンプルで柔軟なサイト生成フレームワーク。<https://github.com/shuding/nextra>
- 学習/テンプレ系も複数プールイン: `vercel/next-learn`・`leerob/next-mdx-blog`・`garmeeh/next-seo`・`unicodeveloper/awesome-nextjs`。

### claude
- **nextlevelbuilder/ui-ux-pro-max-skill**（★112,172）、**gstack**（★125,506・Garry Tan 氏の Claude Code 構成）、**anthropics/skills**（★165,460・公式スキル集）、**Leonxlnx/taste-skill**（★69,720）など、スキル系が引き続き層厚。

### openclaw
- 大型どころが軒並み前週比プラス: **hermes-agent**（★223,371）/ **airi**（★46,182）/ **graphify**（★99,686）/ **cc-switch**（★122,903）。
- 新規: **citrolabs/ego-lite**（★6,981・注目枠。AI エージェント用の高速ブラウザ、ログイン状態を共有できる。openclaw の hermes-agent タグ持ち）。<https://github.com/citrolabs/ego-lite>

---

## 🆕 新規ランクイン（今週プール入り・抜粋）

長寿の定番教材・リファレンス系がまとめて再浮上しました（AI 学習需要の反映と見られます）。

- **openai/whisper**（Python・★106,288）— 大規模弱教師あり音声認識。<https://github.com/openai/whisper>
- **pallets/flask**（Python・★72,011）— Python の軽量 Web フレームワーク。<https://github.com/pallets/flask>
- **trekhleb/javascript-algorithms**（★196,360）/ **yangshun/tech-interview-handbook**（★141,446）/ **jackfrued/Python-100-Days**（★184,745）— アルゴリズム・面接・学習系の定番。
- **tonsky/FiraCode**（★81,875）— プログラミング用リガチャ付き等幅フォント。<https://github.com/tonsky/FiraCode>

---

## まとめ

今週は「AI エージェントをどう強く・安く・並列に動かすか」に人気が集中しました。具体的には ①スキル/ハーネス集（superpowers・ECC・mattpocock/skills）、②トークン削減ゲートウェイ（OmniRoute・rtk・CLIProxyAPI）、③コードのナレッジグラフ化（graphify・codegraph・Understand-Anything）、④並列エージェント運用（orca）の4本柱。openclaw タグの大型リポも堅調でした。来週も同テーマの継続が見込まれます。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
