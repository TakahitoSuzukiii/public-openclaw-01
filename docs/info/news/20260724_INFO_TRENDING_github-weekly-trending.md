# GitHub 週刊トレンド — 2026-07-24

作成日: 2026-07-24 / STATUS: INFO / TOPIC: TRENDING

> 本記事は GitHub 公式 REST API（Representational State Transfer API＝HTTP でデータをやり取りする公式インターフェース）で取得したデータをもとに、週次のトレンド（人気・注目の動き）を初心者にも分かる文体でまとめたものです。掲載基準は原則スター 1万以上・開発者向け。前週（2026-07-17）比でスターが増えたリポジトリ（risers＝上昇株）と、今週あらたに集計対象へ入ったリポジトリ（newcomers＝新顔）を中心に紹介します。
>
> 用語ミニ解説:
> - **スター (star)** … GitHub の「お気に入り」件数。人気・注目度の目安。
> - **リポジトリ (repository, repo)** … ソースコードや文書を保管する場所。1プロジェクト＝1リポジトリが基本。
> - **LLM (Large Language Model＝大規模言語モデル)** … 大量の文章で学習した AI。ChatGPT や Claude の中身。
> - **エージェント (agent)** … LLM に「道具（ツール）を使う手足」を与え、自律的に作業させる仕組み。
> - **MCP (Model Context Protocol＝モデル・コンテキスト・プロトコル)** … AI とツール・データ源をつなぐ共通規格。
> - **スキル (skill / Agent Skill)** … エージェントに追加できる再利用可能な手順書・能力パッケージ。今週の大きな潮流。

---

## 今週のサマリー（先に結論）

- **集計プール 452 リポジトリ**、うち上昇株（risers）334・新顔（newcomers）108。
- **今週の主役は「Agent Skills（エージェント・スキル）」エコシステム。** スキル集・スキル管理ツールが上昇株の上位を独占しています。`mattpocock/skills`（+10,980）、`Graphify-Labs/graphify`（+5,030）、`obra/superpowers`、`msitarzewski/agency-agents` などが軒並み急伸。
- **優先トピック（go / rust / typescript / python / nextjs / claude / openclaw / docker など）はいずれも活況。** とくに **TypeScript × AI エージェント** と **Python × AI アプリ** が濃い週でした。
- 最速の上昇は **`koala73/worldmonitor`（+11,178）** — リアルタイム地政学モニタリングのダッシュボード。

---

## 🚀 今週の上昇株トップ（risers）

### 1. koala73/worldmonitor — +11,178（73,150★・TypeScript）
- **概要:** リアルタイム地政学・インフラ監視ダッシュボード。AI によるニュース集約と状況把握（situational awareness）を1画面に統合。
- **なぜ注目か:** 今週いちばんスターを伸ばした。「Palantir 的な状況把握ツールをオープンソースで」という切り口が刺さっている。OSINT（Open-Source Intelligence＝公開情報インテリジェンス）文脈で話題。
- **主な特徴:** MCP サーバ対応 / ニュース・地政学・インフラ追跡を統合 / ダッシュボード UI。
- **リンク:** <https://github.com/koala73/worldmonitor>

### 2. mattpocock/skills — +10,980（186,555★・Shell）
- **概要:** TypeScript 界の著名人 Matt Pocock 氏の「実務エンジニア向けスキル集」。本人の `.agents` ディレクトリからそのまま公開したもの。
- **なぜ注目か:** 「Agent Skills」潮流の中心。有名人の実運用設定をそのまま覗ける点で人気。
- **主な特徴:** 実エンジニアが日常で使うスキル一式 / コピーして自分のエージェントに導入可能。
- **リンク:** <https://github.com/mattpocock/skills>

### 3. Graphify-Labs/graphify — +5,030（95,184★・Python）【優先: python / claude / openclaw】
- **概要:** コードベース・文書・SQL スキーマ・設定・PDF を、問い合わせ可能な「知識グラフ」に変換するツール。Claude Code / Cursor / Codex / Gemini CLI 用の `/graphify` スキル。
- **なぜ注目か:** ベクトルストア（vector store＝意味検索用DB）を使わず、ローカルの決定論的 AST 解析（Abstract Syntax Tree＝抽象構文木＝コードの構造木）で「全エッジ（関係）を説明可能」にした点が評価。RAG（Retrieval-Augmented Generation＝検索補助生成）の新しい形。
- **主な特徴:** tree-sitter による構文解析 / 知識グラフ生成 / 複数エージェント対応 / OpenClaw トピック付き。
- **リンク:** <https://github.com/Graphify-Labs/graphify>

### 4. earendil-works/pi — +4,887（77,019★・TypeScript）【優先: typescript】
- **概要:** AI エージェント用ツールキット。統一 LLM API・エージェントループ・TUI（Text User Interface＝端末画面の UI）・コーディングエージェント CLI を一式提供。
- **なぜ注目か:** 「自作エージェントの土台」を丸ごと用意でき、コーディング CLI まで含む網羅性が支持されている。
- **リンク:** <https://github.com/earendil-works/pi>

### 5. ruvnet/RuView — +4,844（85,854★・Rust）【優先: rust / typescript / claude】
- **概要:** 市販 WiFi の電波を使い、映像なしで空間認識・バイタル（生体情報）監視・人の在室検知を行う。
- **なぜ注目か:** カメラ無しでプライバシーを保ちつつセンシングという着眼が話題。ESP32 ファームウェア・Home Assistant 連携など IoT 実装が揃う。
- **リンク:** <https://github.com/ruvnet/RuView>

### そのほかの上昇株（抜粋）
- **jamiepine/voicebox** +4,409（46,474★・TS） — オープンソースの AI 音声スタジオ。クローン・書き起こし・生成。
- **rohitg00/ai-engineering-from-scratch** +4,371（43,140★・Python）【優先: rust/ts/python】 — AI エンジニアリングをゼロから学ぶ教材。
- **msitarzewski/agency-agents** +4,117（136,406★・Shell） — 役割特化エージェントを集めた「AI 代理店」セット。
- **codecrafters-io/build-your-own-x** +4,026（531,244★） — 定番技術を自作して学ぶ超人気リスト。
- **obra/superpowers** +3,947（260,552★・Shell） — スキルフレームワーク＋開発方法論。今週のスキル潮流を象徴。
- **NousResearch/hermes-agent** +3,550（219,968★・Python）【優先: python/claude/openclaw】 — 「あなたと共に育つエージェント」。
- **firecrawl/firecrawl** +3,170（155,549★・TS）【優先: typescript】 — Web を検索・スクレイプし LLM 向けに整形する API。

---

## ⭐ 今週の新顔（newcomers・抜粋）

集計プールに新たに入った高スター・活発なリポジトリです。

- **AppFlowy-IO/AppFlowy**（74,249★・Dart） — データ主権を保てる Notion 代替の AI コラボ・ワークスペース。
- **Mintplex-Labs/anything-llm**（63,791★・JS）【優先: hermes-agent】 — ローカルファースト（データを手元に置く）なオールインワン AI エージェント環境。RAG・マルチモーダル対応。
- **coollabsio/coolify**（59,452★・PHP）【優先: nextjs / docker】 — Vercel/Heroku/Netlify 代替の自己ホスト型 PaaS（Platform as a Service）。280種以上をワンクリック配備。
- **appwrite/appwrite**（56,644★・TS）【優先: ts / nextjs / docker】 — Auth・DB・Storage・Functions を備えた BaaS（Backend as a Service）。
- **makeplane/plane**（54,996★・TS）【優先: ts / python / docker】 — Jira/Linear/ClickUp 代替のプロジェクト管理。
- **dockur/windows**（52,562★・Shell）【優先: docker】 — Docker コンテナ内で Windows を動かす。
- **prettier/prettier**（52,119★・JS）【優先: ts / angular】 — 定番コードフォーマッタ。
- **docker/compose**（37,884★・Go）【優先: go / docker】 — 複数コンテナをまとめて定義・起動。
- **aquasecurity/trivy**（37,066★・Go）【優先: go / docker】 — コンテナ/K8s/コードの脆弱性・機密・SBOM スキャナ。

---

## 🎯 優先トピック注目枠

鈴木さんの関心（go / rust / typescript / python / nextjs / claude / openclaw / hermes-agent / docker）で、今週の集計に出た主要リポジトリを言語軸で拾います。

### Claude / OpenClaw / エージェント系
- **anthropics/skills**（163,940★・Python） — Agent Skills の公式リポジトリ。今週の「スキル潮流」の本家。
- **anthropics/claude-code**（138,962★） — 端末で動く Claude のコーディングエージェント。
- **affaan-m/ECC**（232,875★・JS） — エージェントの性能最適化システム（スキル・記憶・セキュリティ）。
- **farion1231/cc-switch**（120,907★・Rust）【rust/ts/claude/openclaw】 — Claude Code / Codex / OpenClaw / Hermes Agent 等をまとめて切り替えるデスクトップ補助ツール。
- **iOfficeAI/OfficeCLI**（21,905★・C#）【claude/openclaw】 — AI エージェント向けに設計された Office（Word/Excel/PowerPoint）操作 CLI。単一バイナリで Office 不要。
- **garrytan/gstack**（124,136★・TS）【ts/claude】 — Garry Tan 氏の Claude Code 設定一式（23ツール）。
- **anomalyco/opencode**（189,368★・TS） — オープンソースのコーディングエージェント。
- **openclaw/openclaw**（384,032★・TS）【rust/ts/openclaw】 — 本家 OpenClaw。自分専用の AI アシスタント。🦞

### Python
- **Shubhamsaboo/awesome-llm-apps**（127,318★・Python） — 100超の AI エージェント/スキル/RAG アプリ集。上昇株（+3,756）。
- **microsoft/markitdown**（168,786★・Python） — 各種文書を Markdown へ変換。
- **github/spec-kit**（123,651★・Python） — 仕様駆動開発（Spec-Driven Development）のツールキット。
- **Comfy-Org/ComfyUI**（122,107★・Python） — ノードベースの拡散モデル（画像生成 AI）GUI。

### TypeScript / Next.js
- **n8n-io/n8n**（197,828★・TS） — AI 対応のワークフロー自動化基盤。
- **langgenius/dify**（150,131★・TS）【ts/python/nextjs/claude】 — エージェントワークフロー・RAG を1つのワークスペースで構築。
- **shadcn-ui/ui**（119,740★・TS）【ts/nextjs】 — 定番の UI コンポーネント集。
- **clash-verge-rev/clash-verge-rev**（133,576★・TS） — Tauri 製のクロスプラットフォーム GUI クライアント。

### Go
- **avelino/awesome-go**（179,101★・Go） — Go のライブラリ・ツール定番リスト。
- **harness/harness**（37,482★・Go）【go/docker】 — SCM・CI/CD・開発環境を統合した開発者プラットフォーム（新顔）。

---

## まとめ

今週の GitHub は「**Agent Skills 元年**」を印象づける週でした。上昇株の上位はほぼスキル集・スキル管理・エージェント基盤で占められ、Anthropic 公式（`anthropics/skills`）から個人の実運用設定（`mattpocock/skills`・`garrytan/gstack`）まで、"エージェントに能力を足す" という発想が一気に主流化しています。優先トピックでは TypeScript のエージェント基盤（pi・firecrawl・opencode）と Python の AI アプリ／変換ツール（awesome-llm-apps・markitdown）が引き続き強い展開。実利用寄りでは自己ホスト系（coolify・appwrite・plane）とコンテナ／セキュリティ（docker compose・trivy）も新顔として存在感を見せました。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
