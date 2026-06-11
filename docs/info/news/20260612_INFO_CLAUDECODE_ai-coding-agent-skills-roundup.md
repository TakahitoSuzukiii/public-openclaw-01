# AIコーディングエージェント／Agent Skills 関連リソース 5選まとめ

> 作成日: 2026-06-12 / STATUS: INFO / TOPIC: CLAUDECODE
> 目的: Claude Code をはじめとする「AIコーディングエージェント」と、その拡張の要となる **Agent Skills（エージェントスキル＝AIに作業手順を自動適用させる仕組み）** に関する注目リソース5件を、初心者にも分かるように要約する。
> 注意: 各リソースの内容・数値は出典記事/リポジトリ公開時点のもの。導入やライセンス（商用可否等）は利用前に各公式で必ず確認すること。

---

## はじめに：なぜ今「Agent Skills」なのか

2026年現在、Claude Code・Cursor・Gemini CLI・Codex など「ターミナルやエディタで動くAIコーディングエージェント」が急速に普及しています。その中で共通の話題になっているのが、**Skills（スキル）** という考え方です。

- **Skill とは**：「APIはこう設計する」「リリースはこの手順で」といった“暗黙知”を `SKILL.md` というファイルに書いておくと、AIが文脈に応じて自動で適用してくれる仕組み。
- 似た仕組みに **Hooks（フック＝特定タイミングで必ず実行されるスクリプト）**、**Subagents（サブエージェント＝独立した文脈で動く別働隊）**、**MCP（Model Context Protocol＝AIと外部ツールをつなぐ標準）** がある。

今回はこの領域の注目リソースを「①ベストプラクティス → ②アプリ雛形集 → ③トークン節約ツール → ④リサーチ用スキル → ⑤開発用スキル集」の順にまとめます。

---

## 1. Claude Code 上級ベストプラクティス 11選（smartscope.blog）

**ひとことで**：Claude Code を毎日使う中〜上級者向けに、公式ベストプラクティスから「仕組みで縛る」運用ノウハウを11個に整理した解説記事。

主なポイント：

- **Hooks は「強制」、CLAUDE.md は「お願い」**：絶対に守らせたい要件（機密ファイルへの書き込み禁止など）は Hooks で確実に。判断が要るルールは CLAUDE.md に。
- **Subagents は「別の文脈窓」で動く**：調査・検証・レビューを本流の文脈から切り離し、失敗の蓄積や偏りが本流を汚染するのを構造的に防ぐ。
- **Skills は文脈で「自動発火」**：明示的に呼ばなくても、状況に応じて該当スキルが自動適用される（`skills/<名前>/SKILL.md` ディレクトリ構成＋YAMLで発火条件を制御）。
- **CLAUDE.md は短く**：長すぎると指示が無視されやすくなる。「消したらAIが間違える行か？」で取捨選択する。“簡潔さは性能要件”。
- **承認疲れ対策**：安全なコマンドは allowlist で許可、OSサンドボックスや `deny` で機密ファイルを隠し、過剰な承認クリックによる安全低下を防ぐ。

**使いどころ**：Claude Code を「なんとなく便利」から「安全に高速運用」へ引き上げたい人の運用設計の参考に。

🔗 出典: [https://smartscope.blog/en/generative-ai/claude/claude-code-best-practices-advanced-2026/](https://smartscope.blog/en/generative-ai/claude/claude-code-best-practices-advanced-2026/)

---

## 2. awesome-llm-apps（Shubham Saboo）

**ひとことで**：そのまま動かせる **AIエージェント／RAGアプリの雛形（テンプレート）100本超** を集めた“クックブック”。

特徴：

- **すぐ動く**：壊れた依存関係なし。3コマンドで起動できる自己完結テンプレート。各テンプレは寄せ集めではなく全てオリジナルで動作検証済み。
- **プロバイダ非依存**：設定変更だけで Claude・Gemini・GPT・Llama・Qwen・xAI を切り替え可能。
- **14カテゴリ**：スターター／上級AIエージェント、マルチエージェント、音声AI、生成UI、ゲームプレイAI、**MCPエージェント**、RAG、**Agent Skills**、メモリ付きLLMアプリ、Chat-with-X、最適化、ファインチューニング、フレームワーク速習など。
- **ライセンスは Apache-2.0**：フォーク・商用利用も可（※利用時に最新ライセンスを確認）。各テンプレに無料チュートリアル（Unwind AI）付き。

**使いどころ**：「RAGパイプラインやエージェントループを毎回ゼロから作る」のをやめ、動く雛形から始めたいとき。

🔗 出典: [https://github.com/Shubhamsaboo/awesome-llm-apps](https://github.com/Shubhamsaboo/awesome-llm-apps)

---

## 3. CodeGraph（colbymchenry）

**ひとことで**：AIコーディングエージェント向けの **事前インデックス済み“コード知識グラフ”**。grep やファイル読み込みの代わりに、構造をグラフで即座に問い合わせさせ、**トークンとツール呼び出しを削減**する MCP サーバ。

特徴：

- **完全ローカル**で動作。Claude Code・Codex・Gemini・Cursor・OpenCode・Antigravity・Kiro・Hermes など主要エージェントを自動設定。
- エージェントがコードベースを探索する際、通常は grep/glob/Read を多数呼んでトークンを消費する。CodeGraph は **シンボル関係・コールグラフ・コード構造** を事前に索引化し、エージェントはファイルをほぼ読まずに回答できる。
- **効果（7言語・7リポジトリでの実測中央値、Opus 4.8 で再検証）**：平均で **コスト16%減・トークン47%減・速度22%向上・ツール呼び出し58%減**。大規模リポほど効果大（例: VS Code でツール呼び出し81%減）。
- 導入：`curl`／`npm` でCLI導入 → `codegraph install`（各エージェントへMCP登録）→ `codegraph init -i`（索引作成）。

**使いどころ**：大きめのコードベースで Claude Code 等のトークン消費・待ち時間が気になるとき。

🔗 出典: [https://github.com/colbymchenry/codegraph](https://github.com/colbymchenry/codegraph)

---

## 4. last30days-skill（mvanhorn）

**ひとことで**：任意のトピックを **Reddit・X・YouTube・TikTok・Hacker News・Polymarket・GitHub など横断で調べ**、「実際の人々の反応（賛同数・いいね・実際のお金）」でスコア付けし、AIが1本のブリーフに要約する **エージェントスキル（`/last30days`）**。

特徴：

- 「編集者を集めるGoogle」ではなく「**人々を検索する**」という発想。各プラットフォームの壁（API・認証）を、自分の鍵・ブラウザセッションを持ち込むことでエージェントが横断する。
- Reddit/HN/Polymarket/GitHub は設定ゼロで即利用可。X・YouTube・TikTok 等はセットアップウィザードで追加。
- 情報源の役割分担が明確：Reddit=本音コメント、X=速報/専門スレ、YouTube=長尺の要点、Polymarket=実際の金で裏付くオッズ、GitHub=PR速度や注目リポ等。
- 導入：Claude Code は `/plugin marketplace add` ＋ `/plugin install`。Codex/Cursor/Copilot/Gemini CLI 等 50以上の Agent Skills ホストは `npx skills add`。OpenClaw でも導入可。

**使いどころ**：変化の速いAI業界の最新動向把握、商談・面談前のリサーチ、何かを作る前の“今みんなが困っていること”調査に。

🔗 出典: [https://github.com/mvanhorn/last30days-skill](https://github.com/mvanhorn/last30days-skill)

---

## 5. agent-skills（Addy Osmani）

**ひとことで**：シニアエンジニアの開発ワークフロー・品質ゲートを **AIコーディングエージェント向けスキルとして体系化** した、本番品質のスキル集。

特徴：

- 開発ライフサイクルに対応した **7つのスラッシュコマンド**：`/spec`（仕様）→ `/plan`（計画）→ `/build`（実装）→ `/test`（検証）→ `/review`（レビュー）→ `/code-simplify`（簡素化）→ `/ship`（出荷）。各コマンドが適切なスキルを自動起動。
- **文脈で自動発火**：API設計中は `api-and-interface-design`、UI構築中は `frontend-ui-engineering` 等が自動で効く。
- `/build` は**計画を一度承認すれば自律実行**（各タスクはテスト駆動＋個別コミット、失敗や危険な箇所では停止）。人手のステップは省くが検証は省かない設計。
- **マルチホスト対応**：Claude Code（推奨・marketplace）、Cursor、Antigravity、Gemini CLI、Windsurf、OpenCode、GitHub Copilot、Kiro、Codex 等。

**使いどころ**：個人/チームで「仕様→出荷」の各段階を、属人化させずAIに一貫した品質で回させたいとき。

🔗 出典: [https://github.com/addyosmani/agent-skills](https://github.com/addyosmani/agent-skills)

---

## まとめ

5件を貫くテーマは、**「AIコーディングエージェントの“運用と拡張”の成熟」** です。

- **運用の型**（①smartscope）：Hooks/Subagents/Skills/allowlist で“仕組みとして縛る”。
- **作り始めの加速**（②awesome-llm-apps）：動く雛形から始める。
- **コスト効率**（③CodeGraph）：索引化でトークン・ツール呼び出しを削減。
- **情報収集の自動化**（④last30days）：人々の反応を横断検索するスキル。
- **開発工程の標準化**（⑤agent-skills）：仕様〜出荷をスキルで一貫運用。

共通キーワードは **Agent Skills / MCP / Subagents**。単体のチャットから、**「スキルとツールで武装したエージェント運用」** へと軸足が移りつつあることが分かります。本サーバ（OpenClaw）でも last30days のように Agent Skills ホストとして導入できるものがあり、運用効率化の選択肢として有力です。
