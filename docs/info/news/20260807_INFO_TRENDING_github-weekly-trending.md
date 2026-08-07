# GitHub 週次トレンド（2026-08-07 版）

作成日: 2026-08-07 / STATUS: INFO / TOPIC: TRENDING

> 前回基準日: 2026-07-31 ／ 集計プール: 449 リポジトリ ／ 掲載基準: スター 10,000 以上・開発者向け（優先トピックは注目枠として例外掲載）。
> データは GitHub 公式 REST API（Search API）から取得。前週スナップショットとの差分で「スター急増（risers）」と「新規ランクイン（newcomers）」を判定しています。

---

## 今週の要点（サマリ）

- 今週の主役は **AIエージェント向け「スキル（skill）」フレームワーク** です。エージェントに“やり方”を教える仕組みが軒並みスター急増しました。ここで言う skill とは、AIコーディングエージェント（Claude Code など）に手順・作法・専門知識を後付けで注入する再利用可能なモジュールのこと。
- スター増加トップは **mattpocock/skills（+10,956）**。以下 **superpowers**、**ponytail**、**taste-skill** など「スキル系」が上位を独占。
- 優先トピックの動き: **claude** 関連が引き続き最多ヒット。**python / typescript** も厚く、**openclaw / hermes-agent** タグの大型リポジトリ（graphify・hermes-agent）も急伸。**go** は DeepSeek 系 CLI が新規ランクイン。

---

## 🚀 スター急増ランキング（risers TOP10）

前週比のスター増加数が大きかったリポジトリです。増加数は「＋」表記。

| 増加 | スター | リポジトリ | 主要言語 | 概要 |
|---|---|---|---|---|
| +10,956 | 208,663 | mattpocock/skills | Shell | 実務者向けスキル集。作者の .agents ディレクトリをそのまま公開 |
| +6,457 | 42,462 | diegosouzapw/OmniRoute | TypeScript | MIT の無料AIゲートウェイ。1エンドポイントで290+プロバイダ集約 |
| +5,213 | 98,204 | DietrichGebert/ponytail | JavaScript | エージェントを「怠惰なシニア開発者」的思考にするスキル |
| +5,009 | 39,587 | stablyai/orca | TypeScript | 並列エージェント群を扱う ADE（Agent Development Environment）|
| +4,940 | 68,306 | Panniantong/Agent-Reach | Python | エージェントに“ネットの目”を与える（X/Reddit/YouTube 読取・検索）|
| +4,301 | 103,987 | Graphify-Labs/graphify | Python | コードやドキュメントを問い合わせ可能なナレッジグラフ化 |
| +4,263 | 268,692 | obra/superpowers | Shell | エージェント向けスキルフレームワーク＆開発方法論 |
| +4,165 | 162,856 | firecrawl/firecrawl | TypeScript | Web検索・スクレイプ・操作を大規模に行うコンテキストAPI |
| +4,094 | 73,814 | Leonxlnx/taste-skill | JavaScript | AIに“センス”を与え、平凡な生成を抑えるスキル |
| +3,817 | 85,268 | earendil-works/pi | TypeScript | 統一LLM API・エージェントループ・TUI を備えたエージェント基盤 |

補足: **anomalyco/opencode（+3,092 / 194,730）** や **microsoft/generative-ai-for-beginners（+3,097 / 116,924）** など定番も引き続き伸びています。

---

## ⭐ 注目リポジトリ詳細

### 1. mattpocock/skills — スキル集の決定版（+10,956）
- **概要**: TypeScript 界隈で著名な Matt Pocock 氏が、自身のエージェント用スキル群（`.agents` ディレクトリ）をそのまま公開したリポジトリ。
- **なぜ注目か**: 今週のスター増加 No.1。実務者が実際に使っている「生きたスキル」を丸ごと参照でき、AIコーディング運用の実例として価値が高い。
- **主な特徴**: 実践的なスキル定義、そのまま流用可能な構成、Shell ベースで軽量。
- **スター**: 208,663（前週比 +10,956）／ **主要言語**: Shell
- リンク: <https://github.com/mattpocock/skills>

### 2. obra/superpowers — スキルフレームワーク＋方法論（+4,263）
- **概要**: エージェント向けの「スキルフレームワーク」と、それを前提としたソフトウェア開発方法論をセットで提供。
- **なぜ注目か**: 累計 26.8 万スターの超大型で、今週も継続的に伸長。単なるスキル集ではなく“開発プロセスごと”提案している点が差別化。
- **主な特徴**: 方法論とツールの一体化、agentic な開発ワークフロー。
- **スター**: 268,692（+4,263）／ **主要言語**: Shell
- リンク: <https://github.com/obra/superpowers>

### 3. firecrawl/firecrawl — Web を丸ごとコンテキスト化（+4,165）
- **概要**: 検索・スクレイプ・ページ操作を大規模に行い、LLM に渡す「コンテキストAPI」。
- **なぜ注目か**: エージェントに最新のWeb情報を与える用途が定番化しつつあり、RAG（Retrieval-Augmented Generation：検索で得た情報を生成に組み込む手法）基盤として採用が拡大。
- **主な特徴**: スクレイピング＋構造化抽出、スケール志向のAPI設計。
- **スター**: 162,856（+4,165）／ **主要言語**: TypeScript
- リンク: <https://github.com/firecrawl/firecrawl>

---

## 🆕 新規ランクイン（newcomers）

今週プールに新たに入った、または再浮上した高スターリポジトリ。

- **juliangarnier/anime**（71,948 / JavaScript）— 定番の JavaScript アニメーションエンジン。CSS/SVG/Canvas を統一的に扱える。<https://github.com/juliangarnier/anime>
- **unslothai/unsloth**（69,685 / Python）— LLM のローカル学習・実行UI。Kimi・Gemma・Qwen・DeepSeek 等に対応。ファインチューニング（微調整）用途で人気。<https://github.com/unslothai/unsloth>
- **hesreallyhim/awesome-claude-code**（51,850 / Python）— Claude Code 向けの厳選リソース集（スキル・エージェント・ツール）。<https://github.com/hesreallyhim/awesome-claude-code>
- **NginxProxyManager/nginx-proxy-manager**（33,803 / TypeScript）— Nginx リバースプロキシをGUIで管理する Docker コンテナ。<https://github.com/NginxProxyManager/nginx-proxy-manager>
- **bojieli/ai-agent-book**（34,300 / Python）— 書籍『AI Agent：設計原理と工程実践』の公開リポジトリ（本文＋PDF＋章別コード）。<https://github.com/bojieli/ai-agent-book>

---

## 🎯 優先トピックの注目枠

NEXUS が重点ウォッチしている技術領域（go / rust / typescript / python / nextjs / claude / openclaw / hermes-agent / angular / docker / nix）の動き。掲載基準（1万スター）未満でも“注目枠”として拾います。

### claude（Claude Code / エージェント関連）
今週最も層が厚いトピック。**ponytail**・**taste-skill**・**Agent-Reach**・**graphify** などスキル/ツール系が軒並み急伸。エージェントの“作法”と“外部接続”を強化する動きが主流です。

### python / typescript
- **Python**: unsloth（ローカル学習UI）、Agent-Reach（Web読取）、ai-agent-book（学習リソース）と、エージェント学習・接続が中心。
- **TypeScript**: OmniRoute（AIゲートウェイ）、orca（並列エージェントADE）、pi（エージェント基盤）、opencode（OSSコーディングエージェント）が伸長。

### go
- **esengine/DeepSeek-Reasonix**（32,878 / Go）— ターミナル常駐型の DeepSeek ネイティブ・コーディングエージェント。prefix-cache（プロンプト先頭部分のキャッシュ）安定性を軸に設計。<https://github.com/esengine/DeepSeek-Reasonix>

### openclaw / hermes-agent
- **Graphify-Labs/graphify**（103,987 / Python, +4,301）— コードベースやドキュメントをナレッジグラフ化。openclaw タグ付きの大型リポジトリ。<https://github.com/Graphify-Labs/graphify>
- **NousResearch/hermes-agent**（227,044 / Python, +3,673）— 「ユーザと共に成長するエージェント」。hermes-agent タグの中核。<https://github.com/NousResearch/hermes-agent>

### nextjs / nix
- **nandorojo/solito**（4,094 / TypeScript）— React Native と Next.js を統一するライブラリ。基準未満だが nextjs 注目枠。<https://github.com/nandorojo/solito>
- **nix-community/nixd**（1,453 / C++）— Nix 言語のランゲージサーバ。基準未満だが nix 注目枠。<https://github.com/nix-community/nixd>

---

## まとめ

今週は「AIエージェント × スキル」の潮流が数字にはっきり表れました。エージェント本体よりも、**エージェントに手順・センス・外部アクセスを与える周辺モジュール**にスターが集中しています。優先トピックでは claude 系が最多、python/typescript が引き続き厚く、openclaw/hermes-agent タグの大型リポジトリも堅調でした。来週も差分ベースで追跡します。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
