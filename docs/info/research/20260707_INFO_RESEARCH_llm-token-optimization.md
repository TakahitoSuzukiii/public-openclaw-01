# LLMトークン最適化ガイド（Claude / Claude Code / OpenClaw ＋ AWS Bedrock / AgentCore）

- **カテゴリ:** INFO / RESEARCH
- **作成日:** 2026-07-07（JST）
- **対象読者:** LLM を実務で使うエンジニア（初心者にも分かる文体）
- **狙い:** トークン消費を抑えつつ効率的に LLM を使うための「王道・ベスト/グッド」と「バッド/アンチ」を切り口ごとに対比で整理する。
- **一次情報:** Anthropic 公式（platform.claude.com）／Claude Code 公式／OpenClaw 公式（ローカル `docs/`）／AWS 公式（docs.aws.amazon.com）。数値は**執筆時点（2026-07）**のもので、料金・下限値は改定され得るため相対値（倍率）中心に記述する。

> ⚠️ **注意:** 料金の絶対額は変動が激しいため本書では書かない。倍率・比率・仕組みで理解し、実額は必ず公式の料金ページで確認すること。マスキング規約に従い、実トークン・鍵・PII は一切記載しない。

---

## 0. まず前提：トークンとコストの基本

**トークン**とは LLM がテキストを扱う最小単位。英語でおおよそ「1トークン ≒ 4文字」（日本語はより多く消費しがち）。課金と処理量は**入力トークン＋出力トークン**で決まる。だから最適化の本質はシンプルで、次の3つに集約される。

1. **入力を減らす** — 毎回送る文脈（システムプロンプト・履歴・ツール定義・添付）を必要最小限にする。
2. **同じ入力を安く再利用する** — Prompt Caching で「変わらない前半」を割引価格で使い回す。
3. **正しいモデルを選ぶ** — 難易度に対して過剰なモデルを使わない。

> OpenClaw の考え方: 「OpenClaw はトークンを（文字ではなく）トークン単位で追跡する。システムプロンプト・会話履歴・ツール呼び出しと結果・添付・要約すべてがコンテキスト枠を消費する」。出典（ローカル）: `docs/reference/token-use.md`。

| | ベスト/グッド | バッド/アンチ |
|---|---|---|
| 発想 | 「毎ターン送るものを削る」を最優先に考える | 「モデルが賢いから全部投げれば良い」と丸投げ |
| 計測 | `/status`・`/usage` 等で実際のトークン内訳を見て改善 | 感覚で最適化した気になる（計測しない） |
| 目標設定 | 入力削減→キャッシュ→モデル選択の順で効く | 出力を削るだけで満足（入力の方が支配的なことが多い） |

---

## 1. コンテキスト設計（Context Engineering）

**考え方:** モデルに渡す文脈は「多いほど賢くなる」わけではない。無関係な文脈はノイズになり、精度もコストも悪化する。**必要な情報だけを、必要なタイミングで**渡す設計（context engineering）が王道。

| 観点 | ベスト/グッド | バッド/アンチ |
|---|---|---|
| 文脈量 | タスクに必要な最小限だけ渡す。要約・抽出して圧縮 | 関連文書を丸ごと全部貼る（"念のため" 全文投入） |
| 静的/動的の分離 | 変わらない前半（指示・定義）と変わる後半（質問）を分離しキャッシュに乗せる | 毎回、静的部分に動的値を混ぜて差し込む（キャッシュ無効化） |
| 履歴 | 古い履歴は要約に畳む。決定事項だけ残す | 全会話を無圧縮で延々と積む |
| 参照 | 大きな資料は「必要時に read/検索で取りに行く」遅延ロード | 事前に全資料を system に埋め込む |
| 反復 | 同じ長い前置きは固定テキスト化して再利用 | 毎回微妙に文言を変えて書き直す（キャッシュ不成立） |

- 一次情報: Anthropic「Context windows」`https://platform.claude.com/docs/en/build-with-claude/context-windows`
- 一次情報: Anthropic「Reduce hallucinations / prompt 設計」`https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview`

---

## 2. Prompt Caching（プロンプトキャッシュ）— 最も効く入力最適化

**仕組み:** リクエストの「安定した前半（システム指示・ツール定義・長い資料）」を**キャッシュ**として保存し、次のリクエストで再計算せず再利用する。読み出しは激安、書き込みは少し割高、という料金構造。

**執筆時点の料金倍率（ベース入力トークン価格に対する相対値）:**

| 種別 | 倍率（対ベース入力） |
|---|---|
| キャッシュ**読み出し**（ヒット） | **0.1x**（＝約1/10） |
| キャッシュ**書き込み**（TTL 5分） | **1.25x** |
| キャッシュ**書き込み**（TTL 1時間） | **2x** |

- **最小キャッシュ長:** モデル依存。例: Opus 4.8 / Sonnet 5 は 1,024 トークン、Haiku 4.5 は 4,096 トークン（下回るとキャッシュされず、エラーにもならない）。
- **ブレークポイント:** 1リクエストにつき最大 **4個**の `cache_control`（`type: ephemeral`, `ttl: "5m"` or `"1h"`）。キャッシュは階層 **`tools` → `system` → `messages`** の順で、前段を変えると後段のキャッシュも無効化される。
- **確認方法:** レスポンスの `cache_read_input_tokens` / `cache_creation_input_tokens` を見る。両方 0 ならキャッシュに乗っていない。

| 観点 | ベスト/グッド | バッド/アンチ |
|---|---|---|
| 配置順 | 安定コンテンツ（ツール定義→システム→長文資料）を**先頭固定**し、変動部分を末尾に | 冒頭に日時・乱数・ユーザー名など毎回変わる値を入れて全キャッシュを壊す |
| TTL 選択 | 5分以内に再利用されるなら 5分キャッシュ（リフレッシュは無償）。間隔が空くエージェント処理は 1時間キャッシュ | 常に 1時間キャッシュ（2x）を選び、実際には5分以内に使い、無駄に高い書込を払う |
| ブレークポイント | 更新頻度の違うブロック境界に置く（例: システムは安定、資料は時々更新） | 意味のない位置に4個乱打／全く使わない |
| 対象 | 長い共通プレフィックス（RAGの固定資料、ツール群、Few-shot例）を乗せる | 短すぎて最小長未満のものをキャッシュしようとする（無効） |

- 一次情報: Anthropic「Prompt caching」`https://platform.claude.com/docs/en/build-with-claude/prompt-caching`
- 一次情報: Anthropic ブログ「token-saving updates（cache-aware rate limits 等）」`https://www.anthropic.com/news/token-saving-updates`

---

## 3. Claude Skills — 段階的開示（Progressive Disclosure）でトークン節約

**仕組み:** Skill は「使い方を教える Markdown（SKILL.md）」。ポイントは、**常時ロードされるのは各 Skill の `name` と `description`（メタデータ）だけ**で、本体の手順は**必要になった時に初めて読み込む**こと。これを progressive disclosure（段階的開示）と呼ぶ。数十のスキルがあっても、常時コストは「見出しリスト」分だけで済む。

- OpenClaw の挙動（ローカル一次情報）: 「システムプロンプトには Skills 一覧の**メタデータのみ**が入り、手順は `read` で必要時にオンデマンド取得。`skills.limits.maxSkillsPromptChars` で上限を制御」。出典: `docs/reference/token-use.md`, `docs/tools/skills.md`。

| 観点 | ベスト/グッド | バッド/アンチ |
|---|---|---|
| description 設計 | 短く的確に「いつ使うか」を書く（トリガー語を含める）。1〜2行 | description に手順本文を全部書く（常時ロードで太る） |
| 本体サイズ | 本体は必要時ロード前提で構造化。詳細は別ファイルに分割し参照 | SKILL.md 1枚に巨大な手順・大量の例を詰める |
| スキル数 | 使うものだけ有効化。エージェント allowlist で絞る | 全スキルを全エージェントに常時展開 |
| 段階化 | 「見出し→本体→補助ファイル」の3段で開示 | 最初から全部を prompt に載せる |

- 一次情報: Anthropic「Agent Skills」`https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview`
- 一次情報（ローカル）: `docs/tools/skills.md`, `docs/tools/skills-config.md`

---

## 4. MCP（Model Context Protocol）— ツール定義のトークンに注意

**問題の核心:** MCP サーバを繋ぐと、その**全ツールのスキーマ（名前・説明・入力JSON Schema）が毎リクエストの入力トークンを消費**する。サーバを増やすほど、実際には数個しか使わなくても常時コストが積み上がる。

**効く工夫:**

- **Tool Search（遅延ロード）:** 全ツールのフルスキーマを事前に載せず、**コンパクトな見出しだけ**をモデルに見せ、必要なツールだけ `describe` して `call` する。OpenClaw は実験的機能として `tools.toolSearch`（`code`/`tools`/`directory` モード）を提供。「大きなカタログを、数個しか使わないと分かっている時に有効」。出典（ローカル）: `docs/tools/tool-search.md`。
- **minimal_output / pagination:** 一覧・検索系ツールは最小出力とページングで結果トークンを抑える（GitHub MCP の公式ガイダンスも「minimal_output=true」「5〜10件バッチ」を推奨）。
- **不要ツールの無効化:** 使わない MCP サーバ／ツールはポリシーで落とす。

| 観点 | ベスト/グッド | バッド/アンチ |
|---|---|---|
| カタログ提示 | 多ツール時は Tool Search で見出しのみ→必要時にスキーマ取得 | 全 MCP のフルスキーマを毎回全投入 |
| スキーマ設計 | 説明・プロパティを簡潔に。冗長な description を避ける | 巨大な JSON Schema／長文 description を全ツールに付ける |
| 出力 | `minimal_output`・ページング（5〜10件）で結果を絞る | 全件・フル詳細を一度に返させる |
| 有効化範囲 | 実際に使うサーバだけ接続 | 「とりあえず全部」接続して常時コスト化 |

- 一次情報: Anthropic「MCP」`https://platform.claude.com/docs/en/agents-and-tools/mcp`
- 一次情報（ローカル）: `docs/tools/tool-search.md`
- 補足（ローカル）: `docs/info/mcp/20260612_INFO_MCP_popular-mcp-and-skills-for-openclaw.md`（当ワークスペースの既存調査）

---

## 5. モデル選択 — 難易度に合ったモデルを使う

**考え方:** 最上位モデル（Opus 系）は賢いが高い。単純な分類・抽出・要約は下位モデル（Haiku 系）で十分なことが多い。**タスクを分解し、難所だけ上位モデル**に回すのが王道。

| タスク例 | 推奨クラス（相対） |
|---|---|
| 難しい推論・設計・長い自律作業 | 最上位（例: Opus 4.8 / Fable 5） |
| 一般的なコーディング・要約・対話 | 中位（例: Sonnet 5） |
| 分類・抽出・定型変換・大量バッチ | 下位・高速（例: Haiku 4.5） |

| 観点 | ベスト/グッド | バッド/アンチ |
|---|---|---|
| 使い分け | サブタスク単位で最小十分なモデルを選ぶ（ルーティング） | 何でも最上位モデルで通す |
| 補助処理 | 要約・圧縮・下処理は安いモデルに委譲 | 要約まで最上位モデルで実施 |
| 評価 | 精度が足りる範囲で下げてコスト実測 | モデル固定で最適化余地を捨てる |

- OpenClaw 補足: **compaction（要約）専用に別モデルを指定可**（`agents.defaults.compaction.model`）。サブエージェントにも安いモデルを割り当て可（`agents.defaults.subagents.model`）。出典（ローカル）: `docs/concepts/compaction.md`, `docs/tools/subagents.md`。
- 一次情報: Anthropic「Models overview」`https://platform.claude.com/docs/en/about-claude/models/overview`
- 補足（ローカル既存調査）: `docs/info/research/20260611_INFO_RESEARCH_claude-fable5-vs-opus48.md`

---

## 6. セッション / メモリ運用 — 長い会話を太らせない

**仕組み（OpenClaw）:** 会話がコンテキスト上限に近づくと**自動コンパクション**が古いターンを要約に畳む。`/compact` で手動実行も可能。要約はディスク上の全履歴を消さず、「次ターンにモデルへ渡す量」だけを減らす。

| 観点 | ベスト/グッド | バッド/アンチ |
|---|---|---|
| 長い会話 | 節目で `/compact`（or 自動）で要約に畳む。決定事項をメモリへ | 1セッションを無限に引き延ばし全履歴を毎回送る |
| 話題転換 | 新しい独立作業は `/new` で新セッション | 無関係な新タスクを長い旧文脈に継ぎ足す |
| 恒久知識 | 重要事実は `MEMORY.md`／メモリツールに保存し本文から外す | 覚えておきたい事を毎回プロンプトに手書きで再投入 |
| メモリ肥大 | `MEMORY.md` は要点のみ・短く。詳細は日次や別ファイルへ | `MEMORY.md` に長文を溜め、毎ターン全読み込み |
| ツール結果 | `toolResultMaxChars` 等で巨大出力を上限内に | ログ全文・巨大 JSON をそのまま文脈に残す |

- OpenClaw の追加ノブ（ローカル一次情報 `docs/reference/token-use.md`）:
  - `agents.defaults.bootstrapMaxChars`（既定 20000）/ `bootstrapTotalMaxChars`（既定 60000）— 起動時注入ファイルの上限。
  - `contextLimits.toolResultMaxChars` / `memoryGetMaxChars` — 実行時の抜粋上限。
  - `imageMaxDimensionPx`（既定 1200）— 画像を縮小して vision トークンを削減。
- 一次情報（ローカル）: `docs/concepts/compaction.md`, `docs/reference/session-management-compaction.md`, `docs/concepts/memory.md`

---

## 7. 出力トークン制御 & Token-Efficient Tool Use

**出力側の最適化:** 出力トークンは入力より単価が高いことが多い。冗長な出力を避けるだけで効く。

| 観点 | ベスト/グッド | バッド/アンチ |
|---|---|---|
| 冗長性 | 「簡潔に」「箇条書きで」「N語以内」等で出力量を制御 | 毎回長い前置き・繰り返し・自己説明を許す |
| 構造化 | 構造化出力（JSON/スキーマ）で必要項目だけ返させる | 自由文で冗長に返させ後段でパース |
| 上限 | `max_tokens` を用途に合わせ適正化 | 常に巨大な `max_tokens` |
| ツール使用 | Claude 4+ は **token-efficient tool use が標準内蔵**（ツール呼び出しのトークンを自動削減）。旧ベータヘッダ `token-efficient-tools-2025-02-19` は不要・無効 | 旧ベータヘッダを付け続ける／ツール往復を無駄に増やす |

- **token-efficient tool use:** 以前はベータヘッダで有効化していたが、**Claude 4+ 系（Opus 4.7+ / Sonnet 5 / Fable 5 等）では常時内蔵**され、ヘッダ指定は不要（付けても無効）。出典: Anthropic 移行ガイド `https://platform.claude.com/docs/en/about-claude/models/migrating-to-claude-4`。
- OpenClaw 補足: **Tokenjuice プラグイン**で `exec`/`bash` の**ノイズの多いツール結果を後処理で圧縮**できる（コマンド自体は変えない）。出典（ローカル）: `docs/tools/tokenjuice.md`。

---

## 8. OpenClaw 固有の運用ノブ（まとめ）

OpenClaw で「毎ターンの固定コスト」を下げる実務ノブ。

| ノブ / 機能 | 効果 | 出典（ローカル docs） |
|---|---|---|
| Skills の progressive disclosure | 手順は必要時のみ read。常時はメタデータのみ | `reference/token-use.md`, `tools/skills.md` |
| `skills.limits.maxSkillsPromptChars` | スキル一覧ブロックの上限 | `tools/skills-config.md` |
| Tool Search（`tools.toolSearch`） | 多ツールを見出し化し遅延ロード | `tools/tool-search.md` |
| Tokenjuice プラグイン | `exec`/`bash` 結果を圧縮 | `tools/tokenjuice.md` |
| 自動 compaction / `/compact` | 長い履歴を要約に畳む | `concepts/compaction.md` |
| `compaction.model` | 要約を安いモデルに委譲 | `concepts/compaction.md` |
| サブエージェント分離＋`subagents.model` | 重い調査を隔離・安いモデルで実行 | `tools/subagents.md` |
| `bootstrapMaxChars`/`bootstrapTotalMaxChars` | AGENTS/SOUL 等の注入上限 | `reference/token-use.md` |
| `contextLimits.toolResultMaxChars` 他 | 実行時抜粋の上限 | `reference/token-use.md` |
| `imageMaxDimensionPx` | 画像縮小で vision トークン削減 | `reference/token-use.md` |
| reasoning off（既定） | 不要な思考トークンを出さない | `tools/thinking.md` |
| グループでの MEMORY 非読込 | 共有チャットで長期メモリを読まない | `reference/token-use.md` |

| 観点 | ベスト/グッド | バッド/アンチ |
|---|---|---|
| 常時コスト | AGENTS.md / SOUL.md / TOOLS.md を簡潔に保つ | 起動ファイルを巨大化し毎ターン注入 |
| ツール | 使うツール/スキルだけ有効化＋多い時は Tool Search | 全 MCP・全スキルを常時フル展開 |
| 思考 | 必要な時だけ thinking を on | 常時 thinking on で思考トークン浪費 |

---

## 9. AWS Bedrock — トークン/コスト最適化

Bedrock で Claude 等を使う場合の最適化観点。仕組みは Anthropic 直と共通点が多いが、**バッチ推論やプロビジョンドスループット**など Bedrock 固有の手段がある。

### 9-1. Prompt Caching（Bedrock 版）
- Converse / ConverseStream / InvokeModel で `cachePoint`（Converse）または `cache_control`（InvokeModel）を指定。**キャッシュ読み出しは低レート、書き込みはやや高レート**、キャッシュ外は通常入力レート。
- **最小トークン/上限はモデル依存**（例: Claude Opus 4.5 / Haiku 4.5 / Sonnet 4.5 は 4,096トークン最小・最大4チェックポイント、Sonnet 4.6 は 1,024最小）。TTL は多くが5分、一部モデルは1時間も可。順序は Anthropic 同様 `tools → system → messages`。
- **注意:** プロンプトキャッシュは**オンデマンド推論のみ対応。バッチ推論 API では非対応**。
- 一次情報: `https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-caching.html`

### 9-2. バッチ推論（Batch Inference）
- 大量・非同期でよいジョブは**バッチで割引レート**。ただしプロンプトキャッシュとは併用不可なので「即時性が要らない大量処理＝バッチ」「反復コンテキスト＝キャッシュ」と使い分ける。
- 一次情報: `https://docs.aws.amazon.com/bedrock/latest/userguide/batch-inference.html`

### 9-3. モデル選択・推論パラメータ・クロスリージョン
- Nova / Claude / その他からタスク難度に合うモデルを選ぶ。`maxTokens` を絞り出力を抑制。クロスリージョン推論で可用性を確保（ただし高需要時はキャッシュ書込が増えることがある）。

### 9-4. プロビジョンドスループット
- **定常的に高スループット**が要るワークロード向け。トークン単価最適化というより「安定した容量確保・レイテンシ保証」の手段。散発的利用にはオンデマンドが安い。
- 一次情報: `https://docs.aws.amazon.com/bedrock/latest/userguide/prov-throughput.html`

| 観点 | ベスト/グッド | バッド/アンチ |
|---|---|---|
| 反復コンテキスト | オンデマンド＋Prompt Caching（安定プレフィックス先頭固定） | 巨大文書を毎回フル送信・キャッシュ未使用 |
| 大量・非同期 | Batch Inference で割引 | 大量ジョブをオンデマンドで即時連打 |
| 定常高負荷 | Provisioned Throughput で容量確保 | 散発利用に Provisioned を常時確保して無駄払い |
| 併用 | 「即時＋反復＝キャッシュ」「非同期大量＝バッチ」を分ける | キャッシュとバッチを混同（バッチはキャッシュ非対応） |
| 計測 | `cacheReadInputTokens`/`cacheWriteInputTokens` で効果検証 | 効果を測らず設定した気になる |

- 一次情報: 料金は `https://aws.amazon.com/bedrock/pricing/`

---

## 10. Amazon Bedrock AgentCore — エージェント運用のトークン/コスト最適化

AgentCore はエージェントの本番運用（メモリ・ツール接続・監視）をマネージドで提供する。トークン観点で重要なのは **Memory** と **Gateway**、**Observability**。

### 10-1. AgentCore Memory（短期／長期）
- **短期メモリ:** 1セッション内のターン単位の文脈を保持。「さっきの話」を毎回プロンプトに手書きで積み直さなくてよい。
- **長期メモリ:** 複数セッションをまたいで**要点（ユーザー嗜好・重要事実・セッション要約）を自動抽出・保存**。次回以降は全履歴ではなく「抽出済みの要点」だけを文脈に載せられる → 長い生ログを毎回送るより**トークン効率が良い**。
- 一次情報: `https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/memory.html`

### 10-2. AgentCore Gateway（ツール/MCP 接続）
- 多数の API/ツールを MCP 互換のゲートウェイに集約。**ツール検索・選択的公開**により、全ツール定義を常時プロンプトに載せずに済ませる設計が可能（MCP 章の Tool Search と同じ発想）。
- 一次情報: `https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/gateway.html`

### 10-3. AgentCore Observability（トークン可視化）
- 各ステップのトークン・レイテンシ・コストを可視化し、**どのプロンプト/ツールが太いか**を特定して削る。「測って削る」を回す土台。
- 一次情報: `https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/observability.html`

| 観点 | ベスト/グッド | バッド/アンチ |
|---|---|---|
| 履歴管理 | 長期メモリの「抽出要約」を文脈に載せる | 全セッションの生ログを毎回全投入 |
| 短期文脈 | 短期メモリでターン参照、必要分だけ | 直近文脈を毎回プロンプトへ手動再構築 |
| ツール | Gateway＋検索でツールを選択的に公開 | 全ツール定義を常時フル展開 |
| 監視 | Observability でトークン内訳を計測→重い箇所を削減 | 可視化せず勘で最適化 |
| メモリ設計 | 保存する事実を絞る（嗜好・決定事項） | 何でも長期メモリに溜め、抽出文脈を肥大化 |

> 補足: AgentCore は当ワークスペースの既存調査にも登場。参照: `docs/info/*/20260629_DONE_AI_agent-execution-platforms-dify-gemini-agentcore.md`。

---

## 11. 早見表（全体総まとめ）

| 切り口 | まず効く一手（ベスト） | 典型的な失敗（アンチ） |
|---|---|---|
| コンテキスト設計 | 必要最小限＋静的/動的を分離 | 念のため全文投入 |
| Prompt Caching | 安定プレフィックスを先頭固定・4breakpoint以内 | 冒頭に可変値でキャッシュ破壊 |
| Claude Skills | description は短く、本体は遅延ロード | SKILL.md に全手順ベタ書き |
| MCP | 多ツールは Tool Search＋minimal_output | 全スキーマ常時フル投入 |
| モデル選択 | 難度別にルーティング、下処理は安いモデル | 何でも最上位モデル |
| セッション/メモリ | compact＋メモリ要点化、話題転換で新セッション | 無限セッション＋MEMORY肥大 |
| 出力制御 | 簡潔指示＋構造化＋max_tokens適正化 | 冗長出力・巨大max_tokens |
| OpenClaw | 起動ファイル簡潔化＋Tokenjuice＋reasoning off | 巨大AGENTS/全ツール/常時thinking |
| Bedrock | 反復=キャッシュ / 非同期大量=バッチ / 定常=Provisioned | 用途を混同・キャッシュとバッチ併用しようとする |
| AgentCore | 長期メモリの抽出要約＋Observabilityで計測削減 | 生ログ全投入・無計測 |

---

## 12. 参考リンク（一次情報）

**Anthropic / Claude**
- Prompt caching: `https://platform.claude.com/docs/en/build-with-claude/prompt-caching`
- Context windows: `https://platform.claude.com/docs/en/build-with-claude/context-windows`
- Token counting: `https://platform.claude.com/docs/en/build-with-claude/token-counting`
- Agent Skills: `https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview`
- MCP: `https://platform.claude.com/docs/en/agents-and-tools/mcp`
- Models overview: `https://platform.claude.com/docs/en/about-claude/models/overview`
- Claude 4 移行ガイド（token-efficient tool use 内蔵化）: `https://platform.claude.com/docs/en/about-claude/models/migrating-to-claude-4`
- ブログ「token-saving updates」: `https://www.anthropic.com/news/token-saving-updates`

**Claude Code**
- Costs / トークン管理: `https://docs.claude.com/en/docs/claude-code/costs`
- `/compact` などコンテキスト運用: `https://docs.claude.com/en/docs/claude-code/overview`

**OpenClaw（ローカル `docs/` を一次情報とする）**
- `reference/token-use.md`（トークン計上・システムプロンプト構成・各種上限）
- `tools/tool-search.md`（Tool Search）／`tools/tokenjuice.md`（結果圧縮）
- `tools/skills.md`・`tools/skills-config.md`（Skills / progressive disclosure）
- `concepts/compaction.md`・`reference/session-management-compaction.md`（要約）
- `concepts/memory.md`・`concepts/context-engine.md`（メモリ・文脈）
- `tools/subagents.md`（サブエージェント分離）／`tools/thinking.md`（reasoning）
- 公開ミラー: `https://docs.openclaw.ai`

**AWS Bedrock / AgentCore**
- Bedrock Prompt caching: `https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-caching.html`
- Bedrock Batch inference: `https://docs.aws.amazon.com/bedrock/latest/userguide/batch-inference.html`
- Bedrock Provisioned Throughput: `https://docs.aws.amazon.com/bedrock/latest/userguide/prov-throughput.html`
- Bedrock 料金: `https://aws.amazon.com/bedrock/pricing/`
- AgentCore Memory: `https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/memory.html`
- AgentCore Gateway: `https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/gateway.html`
- AgentCore Observability: `https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/observability.html`

---

> 本書は個人の学習・運用ノート。数値・仕様は**執筆時点（2026-07）**のもので改定され得る。実額・最新下限は必ず各公式ページで確認すること。機密（トークン・鍵・PII）は本書に一切含めない（マスキング規約準拠）。
