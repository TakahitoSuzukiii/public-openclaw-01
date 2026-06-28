# AI実行基盤（エージェント基盤）調査 — Dify / Gemini Enterprise / Bedrock AgentCore / Strands ほか

- **ステータス:** DONE（調査・記事化完了）
- **作成日:** 2026-06-29（JST）
- **目的:** AI活用による業務改善・効率化・自動化のための「AI実行基盤（AI agent platform）」をキャッチアップする。Dify / Gemini Enterprise を起点に、類似・競合のサービス／フレームワークを整理し、選定の指針を示す。
- **対象読者:** 業務へAIエージェントを導入したいエンジニア。
- **注記:** 本書は各社の**公式ドキュメント／公式ブログ**を一次情報として確認して作成。出典は末尾に明記。製品仕様・名称は更新が速い領域のため、導入時は必ず公式の最新版を再確認すること（特に 2026年4月以降に大型アップデートが多い）。

---

## 0. 結論（先に要点）

- **「AI実行基盤」は一枚岩ではなく、レイヤーで分かれる。** ① エージェントを“組む”フレームワーク層、② “動かす・運用する”ランタイム／プラットフォーム層、③ 業務ユーザが“使う”アプリ層。製品比較はこのレイヤーを揃えないと噛み合わない。
- **Dify** = OSS の「LLMアプリ開発プラットフォーム」。ローコードで RAG・ワークフロー・エージェントを作り、API で組み込む。**自前ホスト可**が最大の強み。
- **Gemini Enterprise（Agent Platform）** = Google の統合エンタープライズ基盤。Vertex AI を発展させ、開発（ADK / Agent Studio）＋運用＋ガバナンス＋業務アプリを1つに束ねる。**Google Cloud Next '26（2026年4月）で発表**。
- **AWS Bedrock AgentCore** = 「どのフレームワーク・どのモデルでも」エージェントを安全に本番運用するための**ランタイム／運用基盤**。Runtime・Memory・Gateway・Identity 等のモジュールを単体でも組合せでも使える。
- **Strands Agents** = AWS 製の OSS **エージェントフレームワーク（SDK）**。「モデル駆動」で数行からエージェントを書ける。
- **「Strands と AgentCore は類似？競合？」への答え** → **競合ではなく補完**。Strands は“組む”層（LangGraph/CrewAI と同列の競合）、AgentCore は“動かす”層。Strands で作って AgentCore で動かす、という縦の関係。詳細は §6。

---

## 1. 用語整理：「AI実行基盤」をレイヤーで捉える

「AIエージェントを業務で動かす」には複数の機能が要る。これを層で分けると製品比較が明快になる。

| レイヤー | 役割 | 代表例 |
|---|---|---|
| **アプリ層**（ユーザが使う） | 非エンジニアがエージェントを発見・作成・共有・実行 | Gemini Enterprise app、Microsoft 365 Copilot |
| **オーケストレーション／開発層**（エージェントを組む） | エージェントのロジック・ツール呼び出し・マルチエージェント連携を定義 | Strands Agents、LangGraph、CrewAI、LlamaIndex、Google ADK、OpenAI Agents SDK |
| **ランタイム／運用層**（動かす・守る） | デプロイ、スケール、セッション分離、メモリ、認証、ツール接続、可観測性 | Bedrock AgentCore、Gemini Enterprise Agent Platform（Runtime 部分） |
| **統合プラットフォーム**（上記を一括提供） | ローコードUI＋RAG＋ワークフロー＋API化を一体提供 | **Dify**、Gemini Enterprise（全体）、Microsoft Copilot Studio、Vertex AI Agent Builder |

> ポイント：Dify と Gemini Enterprise は「統合プラットフォーム」として広い範囲をカバーする。一方 AgentCore は「ランタイム／運用層」に特化（フレームワーク非依存）、Strands は「開発層」のフレームワーク。**比較するときは同じ層同士で比べる。**

補助用語：
- **RAG**（Retrieval-Augmented Generation, 検索拡張生成）：社内文書等を検索してLLMの回答に根拠を与える手法。
- **MCP**（Model Context Protocol）：エージェントとツール／データ源をつなぐオープンなプロトコル。
- **A2A**（Agent-to-Agent）：エージェント同士を連携させるプロトコル。
- **ADK**（Agent Development Kit）：Google のエージェント開発キット。

---

## 2. Dify

OSS の **LLMアプリ開発プラットフォーム**。提供元は LangGenius。

- **位置づけ:** ローコードの統合プラットフォーム。ビジュアルなワークフロー・エディタ上で、RAGパイプライン、エージェント、チャットボットを構築し、Webアプリとして公開、または REST API（Backend-as-a-Service）で自社プロダクトに組み込む。
- **主な機能:** AIワークフロー（GUI）、RAGパイプライン、エージェント機能、モデル管理（複数LLMプロバイダ対応）、可観測性（Langfuse / Opik / Arize Phoenix 連携）、プラグイン／マーケットプレイス。
- **提供形態:**
  - **Dify Cloud:** マネージド。無料の Sandbox プランあり。
  - **Self-host（Community Edition）:** OSS を自社インフラに Docker Compose で数分でデプロイ可能。→ **オンプレ／クローズド環境・データ主権を重視する組織に適合**。
- **得意:** プロトタイプから本番まで素早く到達。エンジニア以外も巻き込める。LLMOps（プロンプト管理・ログ・評価）を一通り内包。
- **留意:** OSS版とクラウド有償版で利用条件が異なる。マルチテナントSaaSとしての再提供にはライセンス上の制約があるため、商用利用時は公式の pricing / ライセンス条項を必ず確認すること。

---

## 3. Gemini Enterprise（Gemini Enterprise Agent Platform）

Google Cloud の**エンタープライズ向け統合AIプラットフォーム**。**Google Cloud Next '26（2026年4月、ラスベガス）で発表**され、従来の **Vertex AI を発展（evolution）**させたもの。

構成要素（公式ブログより）:
- **Gemini Enterprise Agent Platform:** 開発者向け基盤。モデル・開発・チューニングを束ね、エージェントを「build / scale / govern / optimize」する。4つの柱：
  - **Build:** 強化版 **ADK**（グラフベースのサブエージェント編成）、ローコードの **Agent Studio**、**A2A / MCP** 対応。
  - **Scale:** 改良 **Agent Runtime**（高速かつ永続的、エージェント間オーケストレーション）。
  - **Govern / Optimize:** ガバナンス・セキュリティ・最適化。
- **Gemini Enterprise app:** 知識労働者の“正面玄関”。ノーコードの **Agent Designer** でエージェントを自作、またはパートナー製エージェントを利用。コネクタで社内データ・サードパーティ（Oracle / Salesforce / ServiceNow 等）に接続。
- **オープンなパートナーエコシステム:** Google Cloud Marketplace 経由でサードパーティ製エージェントを導入。
- **ガバナンス:** IT部門がエージェントの権限・活動を一元管理する単一コントロールプレーン。ノーコード／プロコード双方を同一のID・セキュリティ・監査モデルで統制（“AIスプロール”の抑制）。

**得意:** Google Workspace / Google Cloud を使う組織で、全社的にエージェントを安全に展開・統制したいケース。開発者からナレッジワーカーまで一気通貫。

**留意:** Google Cloud エコシステム前提。ベンダーロックインと、エンタープライズ契約・コスト構造を要評価。

---

## 4. AWS Bedrock AgentCore

「**どのフレームワーク・どの基盤モデルでも**、エージェントを安全に大規模運用する」ための**エージェント運用プラットフォーム**。インフラ管理不要。

- **位置づけ:** ランタイム／運用層に特化。**フレームワーク非依存**で、CrewAI / LangGraph / LlamaIndex / Google ADK / OpenAI Agents SDK / **Strands Agents** などと組み合わせて使える。モデルも Bedrock 内外（OpenAI / Gemini / Anthropic Claude / Amazon Nova / Meta Llama / Mistral）を問わない。
- **モジュール（単体でも組合せでも利用可）:**
  - **Runtime:** サーバレスのエージェント実行環境。高速コールドスタート、非同期の長時間実行、**真のセッション分離**、組み込みID、マルチモーダル／マルチエージェント対応。MCP / A2A 対応。
  - **Memory:** 短期（マルチターン会話）＋長期（セッションをまたぐ記憶）メモリ。エージェント間で共有可。
  - **Gateway:** 既存の API / Lambda / サービスを **MCP互換ツール**に変換。Salesforce / Zoom / JIRA / Slack 等の既存MCPサーバにも接続。
  - **Identity:** エージェントのID・アクセス・認証管理。Cognito / Okta / Microsoft Entra ID / Auth0 等の既存IdPと互換。
  - **Code Interpreter:** コード実行用の隔離サンドボックス（Python / JavaScript / TypeScript）。
  - **Browser:** エージェント用の安全なブラウザ操作環境。
  - **Observability:** 本番でのエージェント性能・品質の監視。
  - **Harness:** 単一API呼び出しでエージェントループ（モデル・システムプロンプト・ツールをインラインで指定）を定義・実行できるマネージドな仕組み。各セッションは隔離 microVM で動作。
- **得意:** すでに AWS を使い、フレームワークは自由に選びつつ、本番運用のセキュリティ・スケール・統制を AWS に任せたいケース。
- **留意:** これ自体は“エージェントの組み方”を提供しない（＝開発層は別途フレームワークが必要）。AWS 前提。

---

## 5. Strands Agents

AWS 主導の **OSS エージェントフレームワーク（SDK）**。Apache 2.0。Python / TypeScript SDK を提供。

- **コンセプト:** 「**モデル駆動（model-driven）**」。プロンプトとツール定義だけで、複雑なオーケストレーションをモデル自身に委ねる発想。数行でエージェントを書け、ローカル開発から本番まで同じコードでスケール。
- **モデル非依存:** Amazon Bedrock / Anthropic / OpenAI / Gemini を第一級サポート、その他プロバイダ・カスタムも可。
- **内包機能:** エージェントループ、モデルプロバイダ抽象、ツール、コンテキスト管理、実行制限、可観測性。CLI（strandly）、MCPサーバ、Agent Builder などの周辺パッケージも提供。
- **得意:** コードファーストで軽量にエージェントを組みたい開発者。LangGraph / CrewAI の代替候補。
- **AgentCore との関係:** §6 参照（Strands で“組み”、AgentCore で“動かす”）。

---

## 6. 「Strands / Bedrock AgentCore は Dify・Gemini の類似？競合？」

依頼の核心。結論は **レイヤーが違うので“競合”ではなく“補完／一部競合”**。

| 製品 | 主な層 | Dify / Gemini Enterprise との関係 |
|---|---|---|
| **Strands Agents** | 開発（フレームワーク） | **直接競合ではない**。LangGraph / CrewAI / Google ADK と同じ“組む”層の競合。Dify のローコード思想とはアプローチが逆（コードファースト）。 |
| **Bedrock AgentCore** | ランタイム／運用 | **一部競合**。Gemini Enterprise Agent Platform の Runtime / Governance 部分とは競合関係。ただし AgentCore はフレームワーク非依存で“運用層のみ”を切り出す点が異なる。Dify（統合型）とは守備範囲がずれる。 |

縦の関係を図にすると：

```
[ 開発層 ]   Strands / LangGraph / CrewAI / ADK / OpenAI Agents SDK
                         │ ここで「エージェントを組む」
                         ▼
[ 運用層 ]   Bedrock AgentCore（Runtime/Memory/Gateway/Identity…）
                         │ ここで「安全に動かす・スケールする」
                         ▼
[ アプリ層 ] 業務ユーザが利用（社内ツール／チャットUI など）
```

- **Strands × AgentCore は“縦の補完”**：Strands で書いたエージェントを AgentCore Runtime にデプロイ、という典型構成（公式も Strands を AgentCore の対応フレームワークとして明記）。
- **Dify / Gemini Enterprise は“横断（統合型）”**：開発〜運用〜アプリを1製品で提供。AWS 系（Strands＋AgentCore）は「部品を選んで縦に積む」思想。**統合の手軽さ（Dify/Gemini） vs 構成自由度（AWS）** のトレードオフ。

---

## 7. 類似・競合の全体マップ

レイヤー別に主要プレイヤーを整理（業務自動化観点で押さえるべきもの）。

### 7-1. 統合プラットフォーム（Dify と同カテゴリ）
- **Microsoft Copilot Studio:** ノーコードでエージェント／コパイロットを構築、Microsoft 365・Power Platform と統合。Microsoft 環境の組織に強い。
- **Vertex AI Agent Builder（Google）:** Google の従来からのエージェント構築。Gemini Enterprise Agent Platform はこの発展形。
- **Amazon Bedrock（Agents）:** AWS のマネージドなエージェント機能（AgentCore とは別レイヤー／補完関係）。
- **Flowise / Langflow:** OSS のビジュアル LLM フロー構築（Dify に近いローコード系）。
- **n8n:** ワークフロー自動化ツール。AIノードでエージェント的処理も可。既存業務自動化（iPaaS）寄り。

### 7-2. オーケストレーション／開発フレームワーク（Strands と同カテゴリ）
- **LangChain / LangGraph:** 最も普及。LangGraph はグラフベースで状態を持つ複雑なマルチエージェントに強い。運用は LangSmith（可観測性）/ LangGraph Platform。
- **CrewAI:** 役割ベースのマルチエージェント“クルー”を直感的に編成。
- **LlamaIndex:** RAG／データ接続に強み。エージェント機能も拡充。
- **Google ADK:** Gemini Enterprise の中核開発キット（OSSとしても提供）。
- **OpenAI Agents SDK:** OpenAI のエージェント構築SDK（軽量・ツール／ハンドオフ中心）。
- **Microsoft AutoGen / Semantic Kernel:** Microsoft 系のマルチエージェント／オーケストレーション。

### 7-3. ランタイム／運用（AgentCore と同カテゴリ）
- **Gemini Enterprise Agent Platform（Runtime / Governance）**
- **LangGraph Platform（旧 LangServe 系）／LangSmith**
- 各クラウドのサーバレス（Lambda / Cloud Run）上での自前運用

---

## 8. Anthropic（Claude）の位置づけ — Claude Enterprise ほか

依頼により追記（初版では未収録）。Anthropic の提供は**2つの層**に分けて理解する。本記事の主題（AI実行基盤＝エージェントを組む/動かす基盤）に対応するのは後者だが、依頼の「Claude Enterprise」は前者にあたる。

### 8-1. Claude Enterprise（Claude アプリのエンタープライズ・プラン）
**位置づけ:** Dify や Bedrock AgentCore のような「エージェントを開発・運用する基盤」ではなく、**Claude アプリ（claude.ai / claude.com）を組織で安全に使うための最上位プラン**。ナレッジワーカー向けの“アプリ層”の製品で、§1 のレイヤー表では Gemini Enterprise app に近い立ち位置。

主な特徴（公開情報ベース）:
- **セキュリティ／管理:** **SSO（SAML）**、**SCIM** によるユーザー自動プロビジョニング、**ロールベースのアクセス制御**、**監査ログ**、ドメイン管理など、IT 部門向けの統制機能。
- **データの取り扱い:** 入力・出力など**組織のデータをモデル学習に利用しない**ことを既定とする。データ保持・コンプライアンス（SOC 2 等）に配慮した運用。
- **拡張コンテキスト:** 一般プランより**大きなコンテキストウィンドウ**と利用枠（発表当初は 500K トークン枠が訴求された）。
- **コラボレーション:** Projects（プロジェクト単位の文脈共有）、Artifacts、チーム共有など。
- **ネイティブ連携 / Claude Code:** **GitHub 連携**でリポジトリを参照可能。近年は**Claude Code**（エージェント的コーディング CLI）を含むプランも提供され、開発業務の自動化に踏み込んでいる。
- **管理コンソール:** 利用状況の可視化、メンバー・権限管理。

**留意:** これは「AIエージェント開発プラットフォーム」ではなく、エンドユーザー向けの統合アシスタント製品。Dify/Gemini Agent Platform/AgentCore と**同じ土俵で比較するものではない**。ただし Claude Code を介した自動化や、後述の API/Agent SDK と組み合わせることで、業務自動化の入口にはなる。プラン詳細・数値・提供範囲は変動が速いため、導入検討時は公式の料金/エンタープライズページで最新確認が必須。

### 8-2. Claude Developer Platform / Agent SDK / Managed Agents（本記事の主題に対応する層）
本記事のテーマ（§1 の開発層・運用層）に**直接対応する Anthropic の offering** は、Claude Enterprise ではなくこちら:
- **Claude Developer Platform（API / Messages API）:** ツール使用、MCP、構造化出力、コード実行などエージェントの土台。**MCP の提唱元は Anthropic**。
- **Claude Agent SDK:** エージェントを“組む”開発フレームワーク。→ **Strands / LangGraph / CrewAI と同カテゴリ（開発層）**。
- **Managed Agents（ベータ）:** Anthropic がエージェントループとツール実行コンテナ（セッション）を**ホスト**するマネージドな実行基盤（環境・セッション・イベントストリーム・スキル・MCP・メモリ・スケジュール実行）。→ **Bedrock AgentCore / Gemini Agent Platform と同カテゴリ（運用層）**。

要するに「Anthropic で Dify/Gemini/AgentCore に相当するもの」を探すなら、答えは Claude Enterprise ではなく **Claude Agent SDK（開発層）＋ Managed Agents（運用層）** になる。

---

## 9. 選定の指針（業務改善・効率化・自動化の観点）

| もし… | 有力候補 | 理由 |
|---|---|---|
| 自前ホスト／データを外に出せない・小さく早く試す | **Dify（Self-host）** | OSS・Docker で即構築、ローコードで非エンジニアも参加 |
| Google Workspace / GCP 中心、全社展開＋統制重視 | **Gemini Enterprise** | 開発〜業務利用〜ガバナンスを一括、IT統制が標準装備 |
| Microsoft 365 中心 | **Copilot Studio** | M365 / Power Platform と密結合 |
| AWS 中心、フレームワーク自由・本番運用の堅牢性重視 | **Strands ＋ Bedrock AgentCore** | 開発は自由に、運用（分離・認証・スケール）は AWS に委譲 |
| コードファーストで複雑なマルチエージェントを精密制御 | **LangGraph / CrewAI**（＋運用基盤） | グラフ／役割ベースで細かく制御 |
| 既存システム連携・定型業務の自動化が主目的 | **n8n / Dify**（＋MCP/Gateway） | iPaaS 的連携とAIを橋渡し |

実務上の評価軸：① ホスティング（SaaS / 自前 / オンプレ可否）、② モデル選択の自由度、③ 既存システム連携（MCP・コネクタ）、④ ガバナンス・監査・ID統制、⑤ 可観測性・評価（LLMOps）、⑥ ライセンス・コスト、⑦ ベンダーロックイン。

---

## 10. まとめ

- 「AI実行基盤」は**層で理解する**のが最短ルート。Dify・Gemini Enterprise は統合型、AgentCore は運用層、Strands は開発層。
- 依頼の問い（Strands / AgentCore は Dify・Gemini の類似 or 競合か）の答えは、**Strands＝開発フレームワークの競合群、AgentCore＝運用層で一部競合**。両者（Strands＋AgentCore）は縦に積む補完関係であり、統合型の Dify / Gemini とは“設計思想の違い”で対比される。
- 選定は「自社のクラウド／データ要件 × 必要レイヤー」で決まる。まずは小さく Dify（self-host）で PoC、要件が固まったらクラウド統合基盤や AWS 構成へ、という段階導入が現実的。

---

## 11. 出典（公式一次情報・参照日 2026-06-29）

- Dify 公式サイト: https://dify.ai/
- Dify ドキュメント（Introduction）: https://docs.dify.ai/
- Dify GitHub（langgenius/dify）: https://github.com/langgenius/dify
- Gemini Enterprise（新統合／公式ブログ）: https://cloud.google.com/blog/products/ai-machine-learning/the-new-gemini-enterprise-one-platform-for-agent-development
- Gemini Enterprise Agent Platform（公式ブログ）: https://cloud.google.com/blog/products/ai-machine-learning/introducing-gemini-enterprise-agent-platform
- Google Cloud Next '26 ハイライト: https://blog.google/innovation-and-ai/infrastructure-and-cloud/google-cloud/next-2026/
- Amazon Bedrock AgentCore（製品ページ）: https://aws.amazon.com/bedrock/agentcore/
- Amazon Bedrock AgentCore（開発者ガイド / What is）: https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/what-is-bedrock-agentcore.html
- Strands Agents（公式サイト）: https://strandsagents.com/
- Strands Agents（GitHub SDK）: https://github.com/strands-agents
- Claude Enterprise / 料金（Anthropic）: https://www.anthropic.com/pricing ／ https://claude.com/pricing
- Claude（プラットフォーム/Developer Platform・ドキュメント）: https://platform.claude.com/docs
- Claude Code: https://www.anthropic.com/claude-code
- 参考（競合）: LangGraph https://www.langchain.com/langgraph ／ CrewAI https://www.crewai.com/ ／ LlamaIndex https://www.llamaindex.ai/ ／ Microsoft Copilot Studio https://www.microsoft.com/microsoft-copilot/microsoft-copilot-studio ／ Vertex AI Agent Builder https://cloud.google.com/products/agent-builder ／ OpenAI Agents SDK https://platform.openai.com/docs/guides/agents ／ n8n https://n8n.io/ ／ Flowise https://flowiseai.com/

> 注: 競合節の各製品の細部仕様は本調査では一次確認まで行っていない項目を含む（普及済みの一般情報に基づく）。導入検討時は各公式ドキュメントの最新版で再確認すること。
