# GitHub 週次トレンド（2026-06-19 版）

作成日: 2026-06-19 / STATUS: INFO / TOPIC: TRENDING

> 前週（2026-06-12）との差分をもとに、スター 10,000 以上の開発者向けリポジトリから「急上昇（risers）」と「新規ランクイン（newcomers）」を中心にまとめた週次レポートです。優先トピック（go / rust / typescript / python / nextjs / claude / openclaw）は 1 万スター未満でも『注目枠』として別途取り上げます。
>
> 用語メモ:
> - **スター（star）**: GitHub の「お気に入り」数。注目度の目安。
> - **前週比（delta）**: 直近 1 週間で増えたスター数。
> - **MCP（Model Context Protocol / モデル・コンテキスト・プロトコル）**: AI エージェントが外部ツール・データに接続するための共通規格。
> - **スキル（skill）**: AI コーディングエージェント（Claude Code など）に手順・ノウハウを追加注入する設定ファイル群。
> - **RAG（Retrieval-Augmented Generation / 検索拡張生成）**: 外部情報を検索して回答に反映させる手法。

---

## 今週の総評

今週も中心は **AI コーディングエージェント関連**で、特に「スキル（skill）」と「トークン削減」の 2 大テーマが目立ちました。

- **スキル系の総取り**: 急上昇トップは `mattpocock/skills`（前週比 +9,877）。`addyosmani/agent-skills`、`Leonxlnx/taste-skill`、`obra/superpowers` などスキル配布リポが軒並み上昇。Claude Code を中心に「優秀なエンジニアの設定をそのまま使う」流れが続いています。
- **トークン削減（コスト最適化）の波**: `headroom`（出力圧縮）、`rtk`（CLI プロキシで 60〜90% 削減）、`caveman`（“原始人語”で 65% 削減）など、LLM 利用コストを下げるツールが複数同時にランクイン。
- **エージェントの「目と記憶」**: `Agent-Reach`（+8,310）や `Scrapling`、`CloakBrowser` など、エージェントに Web 収集能力を与えるツールが新規登場。`Understand-Anything` / `graphify` のようにコードを知識グラフ化する系統も伸びています。
- **openclaw タグ**: `hermes-agent`、`cc-switch`、`screenpipe`、`open-design` など、OpenClaw 連携を明記するリポが引き続き多数。エコシステムの広がりがうかがえます。

---

## 🚀 急上昇リポジトリ（risers・前週比トップ）

### 1. mattpocock/skills — `+9,877`（136,745★ / Shell）
- **概要**: TypeScript 教育で有名な Matt Pocock 氏が、自身の `.claude` ディレクトリから公開したスキル集。
- **なぜ注目か**: 今週の増加数トップ。「実際のエンジニアが使っている設定」をそのまま導入できる手軽さが支持されています。
- **主な特徴**: Claude Code 向けスキルの実例集。導入は配布ファイルをコピーするだけ。
- **言語 / リンク**: Shell / <https://github.com/mattpocock/skills>

### 2. Panniantong/Agent-Reach — `+8,310`（35,041★ / Python）
- **概要**: AI エージェントに「インターネット全体を見る目」を与える CLI。Twitter / Reddit / YouTube / GitHub / Bilibili / 小紅書（XiaoHongShu）を横断検索。
- **なぜ注目か**: API 利用料ゼロをうたい、エージェントの情報収集を安価に実現。優先トピック python + claude に該当。
- **主な特徴**: 単一 CLI、MCP サーバ対応、各種スクレイパ内蔵。
- **言語 / リンク**: Python / <https://github.com/Panniantong/Agent-Reach>

### 3. iptv-org/iptv — `+7,773`（125,758★ / TypeScript）
- **概要**: 世界中の公開 IPTV（インターネット経由のテレビ放送）チャンネルを集めた m3u プレイリスト集。
- **なぜ注目か**: 開発系とは毛色が違うものの根強い人気で、今週も大きく上昇。
- **主な特徴**: 地域・言語別に整理されたストリーム一覧。
- **言語 / リンク**: TypeScript / <https://github.com/iptv-org/iptv>

### 4. obra/superpowers — `+7,347`（233,263★ / Shell）
- **概要**: エージェント向けスキルフレームワーク兼ソフトウェア開発方法論。
- **なぜ注目か**: 「サブエージェント駆動開発（subagent-driven development）」を掲げ、SDLC（開発ライフサイクル）全体をエージェントで回す思想が話題。
- **主な特徴**: ブレインストーミング〜実装までの一連のスキル群。
- **言語 / リンク**: Shell / <https://github.com/obra/superpowers>

### 5. addyosmani/agent-skills — `+6,680`（63,358★ / Shell）
- **概要**: Google の Addy Osmani 氏による、AI コーディングエージェント向けの実務級スキル集。
- **なぜ注目か**: 著名エンジニア発で信頼度が高く、Claude Code / Cursor / Antigravity IDE などに対応。
- **言語 / リンク**: Shell / <https://github.com/addyosmani/agent-skills>

### 6. Egonex-AI/Understand-Anything — `+5,943`（64,024★ / TypeScript）
- **概要**: 任意のコードを「対話できる知識グラフ」に変換するツール。検索・質問が可能。
- **なぜ注目か**: 「魅せるグラフより教えるグラフ」を掲げ、コード理解を支援。優先トピック typescript + claude に該当。
- **主な特徴**: Claude Code / Codex / Cursor / Copilot / Gemini CLI など幅広く連携。
- **言語 / リンク**: TypeScript / <https://github.com/Egonex-AI/Understand-Anything>

### 7. NousResearch/hermes-agent — `+5,673`（197,585★ / Python）
- **概要**: 「使うほど成長するエージェント」を掲げる Nous Research のエージェント基盤。
- **なぜ注目か**: python + claude + openclaw の 3 トピックに該当。OpenClaw 連携を明記し、エコシステム内で存在感大。
- **言語 / リンク**: Python / <https://github.com/NousResearch/hermes-agent>

### 8. farion1231/cc-switch — `+5,345`（104,690★ / Rust）
- **概要**: Claude Code / Codex / OpenCode / OpenClaw / Gemini CLI / Hermes Agent を一元管理するクロスプラットフォームのデスクトップ補助アプリ。
- **なぜ注目か**: rust + typescript + claude + openclaw の 4 トピック該当。複数エージェントを使い分ける人のハブとして伸長。
- **主な特徴**: Tauri 製、プロバイダ／スキル管理、WSL 対応。
- **言語 / リンク**: Rust / <https://github.com/farion1231/cc-switch>

### 9. pewdiepie-archdaemon/odysseus — `+4,960`（74,233★ / Python）
- **概要**: セルフホスト型の AI ワークスペース。
- **なぜ注目か**: 5/31 作成と新しいながら急伸。自前環境で完結する AI 作業基盤への関心の高さを反映。
- **言語 / リンク**: Python / <https://github.com/pewdiepie-archdaemon/odysseus>

### 10. Leonxlnx/taste-skill — `+4,767`（47,080★ / JavaScript）
- **概要**: AI に「良いセンス」を与え、ありがちで退屈な出力を防ぐスキル。
- **なぜ注目か**: フロントエンド／デザイン品質の底上げを狙う実用スキルとして人気。
- **言語 / リンク**: JavaScript / <https://github.com/Leonxlnx/taste-skill>

その他の上昇株: `multica-ai/andrej-karpathy-skills`（+4,637 / 178,787★・Karpathy 氏の知見を凝縮した CLAUDE.md）、`mvanhorn/last30days-skill`（+4,463 / 44,719★・直近 30 日の話題を横断調査）、`microsoft/markitdown`（+4,022 / 156,036★・各種文書を Markdown 変換）、`affaan-m/ECC`（+3,972 / 218,215★）、`nexu-io/open-design`（+3,916 / 67,877★・ローカル志向の Claude Design 代替）。

---

## ✨ 新規ランクイン（newcomers）

- **cline/cline**（63,534★ / TypeScript）: SDK・IDE 拡張・CLI として使える自律コーディングエージェント。優先トピック typescript。<https://github.com/cline/cline>
- **D4Vinci/Scrapling**（65,007★ / Python）: 単発リクエストから大規模クロールまで対応する適応型 Web スクレイピングフレームワーク。MCP サーバ対応。<https://github.com/D4Vinci/Scrapling>
- **microsoft/ai-agents-for-beginners**（67,574★ / Jupyter Notebook）: AI エージェント構築を学ぶ 12 レッスン。入門に最適。<https://github.com/microsoft/ai-agents-for-beginners>
- **CloakHQ/CloakBrowser**（26,613★ / Python）: ボット検知を回避するステルス Chromium。Playwright のドロップイン代替。<https://github.com/CloakHQ/CloakBrowser>
- **trekhleb/javascript-algorithms**（196,104★ / JavaScript）: 解説付きアルゴリズム・データ構造の定番集。<https://github.com/trekhleb/javascript-algorithms>
- **goldbergyoni/nodebestpractices**（105,353★）: Node.js ベストプラクティス集（2026年7月版）。<https://github.com/goldbergyoni/nodebestpractices>
- **MunGell/awesome-for-beginners**（86,665★）: 初心者向け OSS プロジェクトのまとめ。<https://github.com/MunGell/awesome-for-beginners>

---

## 🎯 優先トピック注目枠（1 万未満含む）

優先トピック（go / rust / typescript / python / nextjs / claude / openclaw）に該当し、特に Next.js 周辺で 1 万スター未満ながら実用的な新顔が登場しました。

- **leerob/next-mdx-blog**（7,555★ / TypeScript）: Next.js + MDX のブログテンプレート。Tailwind CSS・TypeScript・Postgres 構成ですぐ立ち上げ可能。優先トピック typescript + nextjs。<https://github.com/leerob/next-mdx-blog>
- **next-safe-action/next-safe-action**（3,045★ / TypeScript）: Next.js の Server Actions を型安全＆バリデーション付きで扱うライブラリ。zod 連携。優先トピック typescript + nextjs。<https://github.com/next-safe-action/next-safe-action>
- **screenpipe/screenpipe**（19,380★ / Rust）: 画面・音声を 24/7 ローカル記録し、見聞きした内容を AI が把握。OpenClaw / Hermes agent ほか 100+ アプリ連携。優先トピック rust + openclaw。<https://github.com/screenpipe/screenpipe>

トークン削減・コスト最適化系（優先トピック該当）も今週まとまって動きました。
- **chopratejas/headroom**（38,330★ / Python）: ツール出力・ログ・RAG チャンクを LLM 投入前に圧縮し、トークンを 60〜95% 削減。<https://github.com/chopratejas/headroom>
- **rtk-ai/rtk**（63,938★ / Rust）: 開発コマンドのトークン消費を 60〜90% 削減する CLI プロキシ。単一 Rust バイナリ・依存ゼロ。<https://github.com/rtk-ai/rtk>
- **JuliusBrussee/caveman**（74,850★ / JavaScript）: “原始人語”で話すことでトークンを 65% 削減する Claude Code スキル（ネタ要素強め）。<https://github.com/JuliusBrussee/caveman>

---

## まとめ

今週のキーワードは「**スキル**」「**トークン削減**」「**エージェントの情報収集力**」の 3 つ。AI コーディングエージェントを“より賢く・より安く・より広い視野で”動かすためのツール群が、新規・急上昇の両面で席巻しました。OpenClaw タグ付きリポも引き続き多数登場しており、関連エコシステムの裾野が着実に広がっています。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
