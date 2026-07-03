# GitHub 週次トレンド（2026-07-03）

作成日: 2026-07-03 / STATUS: INFO / TOPIC: TRENDING

今週の GitHub トレンドをまとめます。対象は公式 REST API（Application Programming Interface＝プログラムから機能を呼び出す窓口）で取得したスター 10,000 以上・開発者向けのリポジトリです。前週（2026-06-26）との差分から、**スターが急増したもの（risers）**と**新しくランクインしたもの（newcomers）**を中心に紹介します。優先トピック（go / rust / typescript / python / nextjs / claude / openclaw）は 1 万スター未満でも「注目枠」として別途取り上げます。

> 用語メモ: 「スター（star）」= GitHub のブックマーク的な評価で人気の目安。「前週比」= 先週スナップショットとの差分。「リポジトリ（repository）」= ソースコードの置き場所。

---

## 🔥 今週のハイライト：AI エージェント・スキルの熱狂が継続

今週も一貫して目立つのは、**AI コーディングエージェント（AI coding agent＝コードを自動で書く AI）向けの「スキル / 設定集」**です。Claude Code や Codex などに読み込ませて挙動を良くする「CLAUDE.md」や「Agent Skills」系リポジトリが、軒並みスターを伸ばしています。実装フレームワークそのものより、**「エージェントをどう賢く使うか」のノウハウ**に人気が集中している週でした。

---

## 📈 スター急増ランキング（Risers・前週比 Top）

### 1. DietrichGebert/ponytail　`+12,930`（72,914★）
- **概要:** AI エージェントを「部屋で一番怠惰なシニア開発者」のように振る舞わせるスキル。
- **なぜ注目か:** 「最良のコードは書かなかったコード」という YAGNI（You Aren't Gonna Need It＝必要になるまで作るな）思想を AI に注入する発想が受け、今週の増加数トップ。6/12 作成の新顔ながら急伸。
- **主な特徴:** Claude Code / Cursor 向けルール集、プロンプトエンジニアリング。
- **言語:** JavaScript / **リンク:** https://github.com/DietrichGebert/ponytail

### 2. msitarzewski/agency-agents　`+10,423`（126,422★）
- **概要:** フロントエンドから Reddit コミュニティ運用まで、専門特化した「AI エージェント代理店」一式。
- **なぜ注目か:** 各エージェントに人格・プロセス・成果物を持たせる構成で、実務ワークフロー志向のユーザーに刺さっている。
- **言語:** Shell / **リンク:** https://github.com/msitarzewski/agency-agents

### 3. mattpocock/skills　`+8,105`（155,497★）【claude】
- **概要:** TypeScript 界の著名教育者 Matt Pocock 氏の `.claude` ディレクトリ由来のスキル集。
- **なぜ注目か:** 「実務エンジニアのための」実践的スキルとして信頼度が高く、着実に伸長。
- **言語:** Shell / **リンク:** https://github.com/mattpocock/skills

### 4. Panniantong/Agent-Reach　`+7,563`（49,798★）【python / claude】
- **概要:** AI エージェントに「インターネット全体を見る目」を与える CLI。Twitter / Reddit / YouTube / GitHub / Bilibili / 小紅書を API 料金ゼロで読取・検索。
- **なぜ注目か:** 各種 SNS の情報取得を無料でまとめる実用性が評価。MCP（Model Context Protocol＝AI にツールを繋ぐ規格）対応。
- **言語:** Python / **リンク:** https://github.com/Panniantong/Agent-Reach

### 5. obra/superpowers　`+6,079`（245,469★）
- **概要:** エージェント型スキルのフレームワーク兼ソフトウェア開発方法論。
- **なぜ注目か:** 「サブエージェント駆動開発（subagent-driven development）」を掲げ、総スター 24 万超の巨大プロジェクトが依然伸びている。
- **言語:** Shell / **リンク:** https://github.com/obra/superpowers

その他の急増: **JuliusBrussee/caveman**（+5,607 / 82,833★・トークンを 65% 削る"原始人語"スキル）、**headroomlabs-ai/headroom**（+4,324 / 56,224★・LLM 前でツール出力を圧縮しトークン 60〜95% 削減）、**firecrawl/firecrawl**（+4,068 / 143,673★・Web スクレイピング API）。トークン節約と Web アクセスの実用系が堅調です。

---

## ⭐ 優先トピック注目枠

### 【claude / openclaw】エージェント運用の中核
- **NousResearch/hermes-agent**（208,643★, +4,912）: 「あなたと共に育つエージェント」。python / claude / openclaw 全対応で総合力が高い。→ https://github.com/NousResearch/hermes-agent
- **farion1231/cc-switch**（112,828★, +3,825）【rust / typescript / claude / openclaw】: Claude Code / Codex / OpenClaw / Gemini CLI 等を一括管理するクロスプラットフォームのデスクトップ補助ツール。Tauri 製。→ https://github.com/farion1231/cc-switch
- **safishamsi/graphify**（77,000★, +4,459）【python / claude / openclaw】: コード・SQL・ドキュメント等を「問い合わせ可能なナレッジグラフ」に変換するスキル。→ https://github.com/safishamsi/graphify
- **openclaw/openclaw**（381,596★）【typescript / openclaw】: 母体プロジェクト。今週も活発に更新。→ https://github.com/openclaw/openclaw

### 【rust】
- **openai/codex**（95,350★): ターミナル常駐の軽量コーディングエージェント。Rust 実装で堅調。→ https://github.com/openai/codex

### 【typescript / nextjs】
- **anomalyco/opencode**（182,095★): オープンソースのコーディングエージェント。→ https://github.com/anomalyco/opencode
- **vercel/chatbot**（20,573★・newcomer）【nextjs】: Vercel 製のフル機能・改造可能な Next.js AI チャットボット。→ https://github.com/vercel/chatbot
- 注目枠（1 万未満）: **al1abb/invoify**（6,311★・Next.js + shadcn の請求書ジェネレータ）、**saltyshiomix/nextron**（4,426★・Next.js + Electron の定番テンプレ）。

### 【python】
- **github/spec-kit**（117,748★): 仕様駆動開発（Spec-Driven Development）のツールキット。→ https://github.com/github/spec-kit
- **usestrix/strix**（34,447★・newcomer): オープンソースの AI ペネトレーションテスト（脆弱性診断）ツール。→ https://github.com/usestrix/strix

### 【go】
- **openimsdk/open-im-server**（16,495★・newcomer）【go / openclaw】: セルフホスト型のチャット / メッセージング基盤。→ https://github.com/openimsdk/open-im-server

---

## 🆕 新規ランクイン（Newcomers 抜粋）

- **papers-we-love/papers-we-love**（107,544★): コンピュータサイエンスの名論文を読み議論するコミュニティ集。→ https://github.com/papers-we-love/papers-we-love
- **3b1b/manim**（88,058★）【python】: 数学解説動画（3Blue1Brown）で使われるアニメーションエンジン。→ https://github.com/3b1b/manim
- **hesreallyhim/awesome-claude-code**（47,902★）【python / claude】: Claude Code 向けスキル・エージェント・ツールの厳選リンク集。→ https://github.com/hesreallyhim/awesome-claude-code
- **facebook/docusaurus**（65,499★）【typescript】: 保守しやすい OSS ドキュメントサイト生成ツール。→ https://github.com/facebook/docusaurus
- **calesthio/OpenMontage**（32,360★）【python / claude】: AI コーディング助手を「動画制作スタジオ」化するエージェント型システム。→ https://github.com/calesthio/OpenMontage

---

## まとめ

- 今週の主役は引き続き **AI エージェント向けスキル / 設定集**。増加数トップは *ponytail*（+12,930）で、"賢いエージェントの使い方" ノウハウ系が上位を独占。
- 優先トピックでは **claude / openclaw** 周辺（hermes-agent・cc-switch・graphify）が総じて堅調。**トークン節約**（caveman・headroom）と **Web/SNS アクセス**（Agent-Reach・firecrawl）の実用系も伸びています。
- 新規枠は論文・数学・ドキュメント・動画制作と、AI 以外の定番教養系リポジトリも顔を出しました。
- 本格的な差分は次週も継続取得予定です。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
