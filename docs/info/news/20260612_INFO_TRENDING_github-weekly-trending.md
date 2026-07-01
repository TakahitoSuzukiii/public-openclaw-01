# GitHub 週次トレンド（2026-06-12）

> 作成日: 2026-06-12 / STATUS: INFO / TOPIC: TRENDING

前回データ（2026-06-07）との差分から、この1週間でスターが大きく伸びたリポジトリ（risers）と、新たにランクインしたリポジトリ（newcomers）を中心にまとめます。掲載基準はスター 10,000 以上の開発者向けプロジェクトです。加えて、優先的に追っているトピック（go / rust / typescript / python / nextjs / claude / openclaw）は、スター 1 万未満でも「注目枠」として別途取り上げます。

※用語の補足：
- **スター（star）** … GitHub での「お気に入り」数。人気・注目度の目安。
- **前週比** … 前回計測（2026-06-07）からのスター増加数。
- **risers** … スターが大きく増えたリポジトリ（伸び盛り）。
- **newcomers** … 今回のプール（集計対象）に新しく入ってきたリポジトリ。
- **skill / agent skill** … AI コーディングエージェント（Claude Code, OpenClaw など）に手順や知識を持たせる拡張定義のこと。

今週の傾向を一言でいえば、**「AI エージェント向けの skill（スキル）集」が引き続き大量に伸びている**週でした。risers 上位の大半が Claude Code / OpenClaw などのエージェントに組み込む skill・ナレッジグラフ系で占められています。

---

## 🔥 今週の急上昇（risers）

### 1. mvanhorn/last30days-skill（+10,856 → 40,255★）
- **概要**: Reddit・X（旧 Twitter）・YouTube・Hacker News・Polymarket・Web を横断して任意のトピックを調査し、根拠付きの要約を生成する AI エージェント向け skill。
- **なぜ注目か**: 今週の増加幅トップ（約 1.1 万増）。「最近30日（last30days）」という名のとおり、鮮度（recency）を重視した深掘りリサーチを売りにしている。
- **主な特徴**: 複数 SNS・ニュースソースの横断検索、根拠（出典）付きの要約合成。
- **主要言語**: Python ／ **優先トピック**: python, claude, openclaw
- リンク: <https://github.com/mvanhorn/last30days-skill>

### 2. pewdiepie-archdaemon/odysseus（+9,210 → 69,270★）
- **概要**: セルフホスト型（自分のサーバで動かす）の AI ワークスペース。
- **なぜ注目か**: 2026-05-31 作成と新しいにもかかわらず、急速にスターを集めている。約 9,200 増。
- **主要言語**: Python ／ **優先トピック**: python
- リンク: <https://github.com/pewdiepie-archdaemon/odysseus>

### 3. addyosmani/agent-skills（+7,848 → 56,674★）
- **概要**: AI コーディングエージェント向けの「実運用品質（production-grade）」なエンジニアリング skill 集。
- **なぜ注目か**: Google の addyosmani 氏による skill 集で、Claude Code・Cursor・Antigravity などに対応。
- **主要言語**: Shell ／ **優先トピック**: claude
- リンク: <https://github.com/addyosmani/agent-skills>

### 4. mattpocock/skills（+7,010 → 126,868★）
- **概要**: TypeScript 界隈で著名な Matt Pocock 氏の `.claude` ディレクトリ由来の skill 集。
- **なぜ注目か**: 既に 12 万超のスターを持つ大型リポジトリが、さらに週 7,000 増と伸び続けている。
- **主要言語**: Shell ／ **優先トピック**: claude
- リンク: <https://github.com/mattpocock/skills>

### 5. NousResearch/hermes-agent（+6,706 → 191,910★）
- **概要**: 「ユーザーと共に成長する」を掲げる AI エージェント Hermes。
- **なぜ注目か**: 19 万超という超大型でありながら継続して伸びている。Anthropic / Claude / OpenClaw など主要エコシステムに対応をうたう。
- **主要言語**: Python ／ **優先トピック**: python, claude, openclaw
- リンク: <https://github.com/NousResearch/hermes-agent>

### その他の急上昇（抜粋）
- **Leonxlnx/taste-skill**（+6,606 → 42,313★, Shell, claude）: AI に「センス（taste）」を持たせ、ありきたりな出力を防ぐ skill。
- **harry0703/MoneyPrinterTurbo**（+6,000 → 86,738★, Python）: AI で短尺動画をワンクリック生成。
- **obra/superpowers**（+5,914 → 225,915★, Shell）: エージェント skill のフレームワーク兼ソフト開発方法論。今回プール内で最大級のスター数。
- **farion1231/cc-switch**（+5,524 → 99,345★, Rust）: Claude Code / Codex / OpenClaw / Gemini CLI などをまとめて切り替えるデスクトップ補助ツール（Tauri 製）。
- **safishamsi/graphify**（+5,309 → 66,256★, Python）: コードや SQL スキーマ等を問い合わせ可能なナレッジグラフに変換する skill。
- **microsoft/markitdown**（+5,206 → 152,013★, Python）: 各種ファイル・Office 文書を Markdown へ変換する Microsoft 製ツール。
- **colbymchenry/codegraph**（+4,758 → 48,176★, TypeScript, claude）: 事前インデックス化したコードのナレッジグラフ。100% ローカル動作でトークン削減。

---

## 🆕 新規ランクイン（newcomers）

今回プールに新しく入ってきた、スター 1 万以上のリポジトリです。

- **react/react**（245,805★, JavaScript）: 言わずと知れた Web/ネイティブ UI ライブラリ。
- **massgravel/Microsoft-Activation-Scripts**（178,172★, Batchfile）: Windows / Office のオープンソース アクティベータ。
- **react/react-native**（126,005★, C++）: React でネイティブアプリを作るフレームワーク。
- **MisterBooo/LeetCodeAnimation**（76,593★, Java）: LeetCode の問題をアニメーションで解説。
- **Eugeny/tabby**（72,086★, **TypeScript**）: モダンなターミナル（SSH / シリアル / Telnet 対応）。
- **openinterpreter/openinterpreter**（63,883★, **Python**）: Deepseek・Kimi・Qwen などオープンモデル向けの軽量コーディングエージェント。
- **Egonex-AI/Understand-Anything**（58,080★, **TypeScript**, claude）: 任意のコードを対話的に探索できるナレッジグラフ化ツール。
- **heygen-com/hyperframes**（27,089★, **TypeScript**）: 「HTML を書けば動画になる」、エージェント向けの動画生成フレームワーク。
- **hugohe3/ppt-master**（26,982★, **Python**）: 任意の文書から編集可能な本物の PowerPoint を AI 生成。
- **Panniantong/Agent-Reach**（26,731★, **Python**, claude）: AI エージェントに Twitter / Reddit / YouTube / GitHub 等の読取・検索能力を与える CLI。
- **teng-lin/notebooklm-py**（16,299★, **Python**, claude, openclaw）: Google NotebookLM の非公式 Python API ＆ エージェント skill。
- **vercel/commerce**（14,080★, **TypeScript**, **nextjs**）: Next.js 製の EC（電子商取引）スターターキット。

---

## ⭐ 注目枠（優先トピック・スター 1 万未満も含む）

優先的に追っているトピックに該当するもののうち、上記に載らなかったものを拾います。

- **iamvishnusankar/next-sitemap**（3,736★, **TypeScript**, **nextjs**）: Next.js 向けのサイトマップ／robots.txt 生成ツール。静的・動的・SSR ページに対応。nextjs エコシステムの定番ユーティリティで、今回 newcomers として観測されました。

### 言語別トピックの動き（priorityHits より抜粋）
- **Go**: `syncthing/syncthing`（85,265★）— オープンソースの継続的ファイル同期（P2P）。
- **Rust**: `zed-industries/zed`（85,092★）、`astral-sh/uv`（86,318★ / Rust 製の高速 Python パッケージ管理）。
- **Python**: `vllm-project/vllm`（82,712★ / LLM 推論サーバ）、`PaddlePaddle/PaddleOCR`（82,010★）、`infiniflow/ragflow`（82,574★ / RAG エンジン）。
- **TypeScript**: `anomalyco/opencode`（173,692★ / オープンソースのコーディングエージェント）、`Stirling-Tools/Stirling-PDF`（80,709★）、`lobehub/lobehub`（78,573★）。
- **Claude / OpenClaw 関連**: `anthropics/skills`（149,920★ / Anthropic 公式の Agent Skills 集）、`thedotmack/claude-mem`（81,992★ / セッション横断の永続メモリ）、`nexu-io/open-design`（63,959★ / ローカルファーストなデザインツール）。

---

## まとめ

- 今週も **AI エージェント向け skill 集・ナレッジグラフ系の伸びが圧倒的**。risers 上位 10 件中、多くが Claude / OpenClaw エコシステム関連でした。
- 優先トピックでは **Python と Claude/OpenClaw の重なり**が特に厚く、リサーチ系（last30days-skill）・コードグラフ系（graphify, codegraph）・メモリ系（claude-mem）が目立ちました。
- newcomers には react 系・react-native といった定番大型に加え、**動画/スライド生成（hyperframes, ppt-master）** や **NotebookLM 連携（notebooklm-py）** など、エージェントに「成果物を作らせる」方向の新顔が入ってきています。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
