# 20260614_INFO_RESOURCES_ai-agent-llm-learning-links.md - AIエージェント／LLM 学習リソース集（OpenClaw・Claude・LangChain ほか）

> STATUS: INFO / CATEGORY: research / 作成日: 2026-06-14
> AIエージェント開発・LLM 学習・OpenClaw／Claude 活用に役立つ公開リポジトリ／公式ドキュメントをカテゴリ別に整理したリンク集（キュレーション）。
> 各項目は「何か（What）」「なぜ役立つか（Why）」を一言添えた。出典 URL は末尾にまとめて記載。

## このドキュメントの使い方

- **目的**: 散在しがちな良質な一次情報（公式・著名 awesome リスト・学習コース）を 1 か所に集約し、学習や検証の入口にする。
- **対象読者**: これから AIエージェント／生成AI を学ぶ人、OpenClaw / Claude Code を業務・個人開発で使いたい人。
- **おすすめの読む順（初学者）**: ① LLM の基礎（`mlabonne/llm-course`）→ ② 生成AI 全体像（`awesome-generative-ai-guide`）→ ③ Agent Skills の考え方（`awesome-agent-skills`）→ ④ 実装（`claude-cookbooks` / `claude-quickstarts` / Strands コース）→ ⑤ フレームワーク（LangChain / LangGraph）→ ⑥ OpenClaw でスキル/ユースケースを試す。

---

## 1. OpenClaw（本体・スキル・ユースケース）

| リソース | 何か | なぜ役立つか |
|---|---|---|
| **openclaw/openclaw** | 自分のデバイス上で動かす個人向け AI アシスタント本体（「The lobster way 🦞」）。WhatsApp・Telegram・Slack・Discord・Signal・iMessage など多数のチャネルに対応し、音声入出力や操作可能な Canvas も持つ。 | 本環境（NEXUS）の土台。インストール／オンボーディング（`openclaw onboard`）から公式ドキュメントまでの起点。 |
| **VoltAgent/awesome-openclaw-skills** | OpenClaw 公式スキルレジストリ（ClawHub）から **5,400+ のスキルを精査・カテゴリ分類**した awesome リスト。スパム／重複／低品質／悪性を除外済み。 | 「やりたいこと」に合うスキルを探す・インストールする際のカタログ。ユースケースの発想源にもなる。 |
| **hesamsheikh/awesome-openclaw-usecases** | OpenClaw を日常でどう使っているかの **ユースケース集**（コミュニティ収集）。 | スキル単体ではなく「生活・業務をどう改善するか」の実例集。導入の壁＝使い道探しを解消する。 |

## 2. Claude / Anthropic（公式・周辺ツール）

| リソース | 何か | なぜ役立つか |
|---|---|---|
| **anthropics/claude-quickstarts** | Claude API で**デプロイ可能なアプリ**を素早く作るためのプロジェクト集（カスタマーサポートエージェント、金融データアナリスト 等）。 | ゼロから作らず、動く土台を改造して学べる。エージェント実装の型を掛むのに最適。 |
| **anthropics/claude-cookbooks** | Claude 活用の**レシピ集（ノートブック）**。コピーして使えるコード断片で、ツール使用・RAG・ビジョン等を網羅。 | 「この機能どう書く？」の逆引き。実装の手本として再利用しやすい。 |
| **Alishahryar1/free-claude-code** | Claude Code の Anthropic Messages API トラフィックを**任意プロバイダ（無料／有料／ローカル、17 種）へルーティングするプロキシ**。モデル別ルーティング、`/model` ピッカー対応、Discord/Telegram ラッパーや音声書き起こしも。 | Claude Code を多様なモデルで試す・コストを抑える検証に。仕組み（プロキシ設計）の学習素材としても有用。※利用は各プロバイダ規約・自社ポリシーに従うこと。 |

## 3. AIエージェント開発 / Agent Skills

| リソース | 何か | なぜ役立つか |
|---|---|---|
| **aws-samples/sample-getting-started-with-strands-agents-course** | **Strands Agents** フレームワークで AIエージェントを学ぶ 4 段階コース（基礎 → MCP/Hooks/セッション → マルチエージェント → 本番デプロイ）。Bedrock/Anthropic 連携、MCP・A2A、LangFuse/RAGAS による評価を扱う。無料の動画講座付き。 | ハンズオンで「基礎から本番」まで体系的に学べる。AWS／Bedrock 環境との親和性が高い。 |
| **skillmatic-ai/awesome-agent-skills** | **Agent Skills（`SKILL.md`）**の決定版リソース集。段階的開示（progressive disclosure）でコンテキストを節約する設計思想・作り方・ベンチマークを段階別に整理。 | OpenClaw / Claude のスキル設計の背景理論と実践がまとまっている。スキル自作時の指針。 |

## 4. 生成AI / LLM 学習ロードマップ

| リソース | 何か | なぜ役立つか |
|---|---|---|
| **aishwaryanr/awesome-generative-ai-guide** | 生成AI の**総合ガイド**。論文まとめ、学習コース、面接対策などを継続的に集約。 | 生成AI 全体の地図。最新動向の追跡と体系的キャッチアップに。 |
| **mlabonne/llm-course** | 人気の **LLM 学習コース**。「Fundamentals / Scientist / Engineer」のロードマップと Colab ノートブックで実習。 | 理論と実装をバランス良く学べる定番。学習順路に迷ったらまずこれ。 |

## 5. LangChain / LangGraph

| リソース | 何か | なぜ役立つか |
|---|---|---|
| **LangChain 公式 Overview**（docs.langchain.com） | `create_agent` を中心とした**最小・高構成性のエージェントハーネス**。モデル＋ツール＋プロンプト＋ミドルウェアを組み合わせて構築。OpenAI/Anthropic/Google 等に対応。 | エージェント実装の標準的フレームワークの公式入口。「Agent = Model + Harness」の考え方が学べる。 |
| **kyrolabs/awesome-langchain** | LangChain エコシステム（ツール・プロジェクト・テンプレート）の**キュレーション**。 | 周辺ツールの全体像把握と、車輪の再発明回避に。 |
| **langchain-ai/langgraph** | **ステートフルなマルチアクター**アプリをグラフで構築するライブラリ。分岐・ループ・人間の介在（HITL）・永続化を表現しやすい。 | 複雑なエージェント／ワークフローを堅牢に組む際の中核。本環境の HITL 設計とも相性が良い。 |
| **LangGraph 公式 Overview**（docs.langchain.com） | LangGraph の公式ドキュメント。 | グラフベース設計の正確な仕様・パターンの一次情報。 |

## 6. トレンドウォッチ

| リソース | 何か | なぜ役立つか |
|---|---|---|
| **GitHub Trending（monthly）** | GitHub の**月間トレンドリポジトリ**一覧（言語フィルタ可）。 | 新しい OSS／エージェント関連プロジェクトの定点観測に。継続キャッチアップの習慣づけに有効。 |

---

## 本環境（OpenClaw / NEXUS）との関係

- **スキル**: 1 章のリストから ClawHub 経由で `openclaw skills install <slug>` 等で導入できる。`~/.openclaw/skills/`（グローバル）または `<project>/skills/`（ワークスペース）に配置。
- **設計思想**: 3 章の Agent Skills（段階的開示）や 5 章の LangGraph（HITL・永続化）は、本環境で重視する「最小構成・HITL・コンテキスト節約」と方向性が一致する。
- **学習 → 検証**: 学んだ内容は本環境のサンドボックスで小さく試し、有用なら手順をドキュメント化する運用と相性が良い。

## 出典（URL）

- OpenClaw 本体: https://github.com/openclaw/openclaw
- awesome-openclaw-skills: https://github.com/VoltAgent/awesome-openclaw-skills
- awesome-openclaw-usecases: https://github.com/hesamsheikh/awesome-openclaw-usecases
- GitHub Trending（monthly）: https://github.com/trending?since=monthly
- free-claude-code: https://github.com/Alishahryar1/free-claude-code
- claude-quickstarts: https://github.com/anthropics/claude-quickstarts
- claude-cookbooks: https://github.com/anthropics/claude-cookbooks
- Strands Agents コース: https://github.com/aws-samples/sample-getting-started-with-strands-agents-course
- awesome-agent-skills: https://github.com/skillmatic-ai/awesome-agent-skills
- awesome-generative-ai-guide: https://github.com/aishwaryanr/awesome-generative-ai-guide
- llm-course: https://github.com/mlabonne/llm-course
- LangChain 公式 Overview: https://docs.langchain.com/oss/python/langchain/overview
- awesome-langchain: https://github.com/kyrolabs/awesome-langchain
- langgraph（リポジトリ）: https://github.com/langchain-ai/langgraph
- LangGraph 公式 Overview: https://docs.langchain.com/oss/python/langgraph/overview

> 注: 各リポジトリ／ドキュメントの内容・名称は更新されることがある。最新は出典 URL を参照。外部リンクの利用は各サイトの規約・自社ポリシーに従うこと。
