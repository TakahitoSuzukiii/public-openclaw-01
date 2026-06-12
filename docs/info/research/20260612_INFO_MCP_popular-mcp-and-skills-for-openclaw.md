# 世界的に人気の MCP と Agent Skills 入門 — OpenClaw での活用ガイド

> 作成日: 2026-06-12 / STATUS: INFO / TOPIC: MCP
> 目的: 「MCP（Model Context Protocol）」と「Agent Skills（エージェントスキル）」のうち、世界的に人気のものを初心者にも分かるように整理し、本サーバ（OpenClaw）での活用方法・導入の方向性をまとめる。
> 関連: ニュース記事 `docs/info/news/20260612_INFO_CLAUDECODE_ai-coding-agent-skills-roundup.md`、導入手順 `docs/openclaw/002_DONE_SETUP_mcp-setup-guide.md`。
> 注意: 人気度・対応状況は出典（2026年時点）に基づく。各サーバ/スキルの導入・利用規約・認証スコープは利用前に必ず確認すること。

---

## 0. まず用語：MCP・Skills・Hook・Subagent の違い

| 用語 | ひとことで | 役割 |
|---|---|---|
| **MCP（Model Context Protocol）** | AIと外部ツール/データをつなぐ「共通プラグイン規格」 | GitHub・Slack・DB 等の機能を *ツール* としてAIに提供。1つ作れば多数のAIから使える |
| **Agent Skills** | AIに「作業手順・専門知識」を自動適用させる仕組み | `SKILL.md` に手順を書くと、文脈に応じて自動発火（例: Word作成、レビュー手順） |
| **Hook** | 特定タイミングで「必ず」実行されるスクリプト | 保存後の自動lint、機密ファイル書込みブロック等。確実性を担保 |
| **Subagent** | 独立した文脈で動く「別働隊」 | 調査・検証・レビューを本流から切り離し、汚染や偏りを防ぐ |

> MCP は「AIにできること（道具）を増やす」、Skills は「AIのやり方（手順・知識）を揃える」。補い合う関係です。MCP は 2024年末に Anthropic が公開したオープン標準で、2026年時点で Claude・ChatGPT・Cursor・Windsurf・Cline、さらに **OpenClaw を含む主要エージェント基盤がネイティブ対応**しています（出典: Cubitrek）。

---

## 1. 世界的に人気の MCP サーバ

「1つ統合を書けば対応する全AIから使える」のが MCP の強み。2026年時点で本番投入が多い代表例をカテゴリ別に挙げます（出典: Cubitrek / PopularAiTools / cybersecuritynews 等）。

### コード・バージョン管理
- **GitHub MCP（公式）** — 2026年で**最もインストールされている** MCP。Issue取得・ブランチ作成・PR作成・コードレビュー応答までエージェント内で完結。※本サーバでも導入済み。
- **GitLab MCP（公式）** — GitLab 版。同じエージェントコードで接続先だけ差し替え。
- **Linear MCP（公式）** — チケットのトリアージ・ステータス更新。GitHub MCP と組み合わせて開発ループを自動化。

### コミュニケーション
- **Slack MCP** — 2026年で**2番目に人気**。朝会レポート・サポート一次対応・オンコール対応エージェントの定番。
- **Discord MCP（コミュニティ）** — チャンネル/メッセージ/スレッド/ロール操作。コミュニティ運営・サポート向け。※本サーバは Discord 連携が主経路。
- **Gmail / Google Workspace MCP（Google公式・beta）** — メール仕分け・カレンダー予定作成・Drive検索。非エンジニアにも刺さる高ROI統合。

### CRM・営業
- **HubSpot MCP（公式）** — 2026年で最も使われる CRM MCP。リード強化・商談更新・パイプライン整備。
- **Salesforce MCP（公式）** — エンタープライズ営業向け。SOQL クエリ・各種オブジェクト操作。
- **Attio MCP（公式）** — スタートアップ向けの柔軟なデータモデル。

### データベース・データ基盤
- **Postgres MCP（Anthropic リファレンス）** — SQL クエリ（既定は読み取り専用）・スキーマ探索。
- **SQLite / その他DB MCP** — 軽量DBやデータ分析用途。

### Web・ブラウザ・検索・その他人気どころ
- **Filesystem MCP（公式リファレンス）** — ローカルファイルの読み書き。最も基本的な土台。
- **Fetch MCP（公式リファレンス）** — URL取得しAI可読なテキスト化。
- **Brave Search MCP** — Web検索をツール化。
- **Playwright / Puppeteer MCP** — ブラウザ自動化（操作・スクショ・スクレイピング）。※本サーバでも Playwright MCP 導入済み（別記事で詳説）。
- **Stripe / Sentry / Figma / Notion / Cloudflare / AWS MCP** — 決済・エラー監視・デザイン・ドキュメント・インフラ。公式提供が増加中。
- **Sequential Thinking / Memory MCP（公式リファレンス）** — 思考の構造化や簡易メモリ。

> 選び方の目安: ①公式提供があるか ②自分の使うサービスに対応するか ③self-host か hosted か（社内データやSSO配下は self-host が無難）。「toy デモ」も多いので、本番は公式・実績あるものを選ぶ。

---

## 2. 世界的に人気の Agent Skills

Skills は「AIに専門手順を持たせる」仕組み。2026年時点では公式チーム製のものが急増しています（出典: VoltAgent/awesome-agent-skills は **1400件超**を収録、Claude Code・Codex・Gemini CLI・Cursor・Copilot 等と互換）。

### Anthropic 公式スキル（特に定番）
- **docx / pptx / xlsx / pdf** — Word・PowerPoint・Excel・PDF の作成/編集/解析。「資料作成の自動化」に直結。
- **doc-coauthoring** — ドキュメントの共同編集。
- **mcp-builder / skill-creator** — MCP サーバや新スキル自体を作るためのメタスキル。

### 開発元チーム製スキル（一例）
- **Stripe / Vercel / Cloudflare / Netlify / Sentry / Figma / HashiCorp(Terraform) / MongoDB / Firebase / Hugging Face / Notion** など、各社が自社サービスのベストプラクティスをスキル化。
- **Trail of Bits（セキュリティ）** — セキュアコーディング/監査手順。
- **Addy Osmani（Web Quality）** — Web 品質・パフォーマンス改善手順。

### コミュニティで人気
- **Superpowers** / **ui-ux-pro-max**（UI/UX強化）など、開発生産性系が上位（出典: scriptbyai）。
- **last30days**（Reddit/X/YouTube等を横断調査するリサーチスキル）。

> 導入の考え方: スキルは「毎回プロンプトに書いていた手順」を*知識レイヤー*として常設するもの。資料作成（Office系）・コードレビュー・セキュリティ・リサーチなど、**繰り返す定型作業**から入れると効果的。

---

## 3. OpenClaw での活用方法（具体策）

本サーバは MCP ネイティブ対応で、すでに GitHub / Playwright / Excel / PowerPoint / AWS / drawio などの MCP を導入済みです。さらに広げる方向性:

### すぐ効く活用アイデア
1. **資料作成の自動化**: Excel/PowerPoint MCP ＋ Anthropic の docx/pptx/xlsx スキルで、調査記事やデータからスライド/表を自動生成（#53 で実証済み）。
2. **開発ループ自動化**: GitHub MCP ＋ レビュー/簡素化スキルで、Issue→ブランチ→PR→レビューを半自動化。
3. **情報収集の定常化**: Fetch / Brave Search MCP ＋ リサーチ系スキルで、週次トレンドやセキュリティ情報を要約（既存の週次 cron と親和）。
4. **ブラウザ業務**: Playwright MCP で、ログイン確認・スクショ・定型操作（別記事 #58 で詳説）。

### 導入手順（概略）
- MCP の追加は `openclaw.json` の MCP 定義に server を足す（コマンド/引数/env）。詳細は `docs/openclaw/002_DONE_SETUP_mcp-setup-guide.md`。
- Skills は対応ホストにインストール（Claude Code 系なら `/plugin marketplace add ...` / `npx skills add ...`）。OpenClaw でも Agent Skills 対応のものは利用可。
- **セキュリティ**: 認証トークンは `.env` 等に置き、ドキュメント/ログに実値を残さない（本サーバのマスキング規約に従う）。self-host か hosted かは扱うデータの機微度で判断。

### Hook / Subagent の併用
- **Hook**: 機密ファイル（`.env`/`.key`/`.pem`）への書込みブロック、保存後の自動整形などを「確実」に効かせる。
- **Subagent**: 大規模調査やレビューを別文脈に逃がし、本流の文脈汚染を防ぐ。

---

## 4. まとめ・次アクション提案

- **MCP = 道具を増やす / Skills = やり方を揃える / Hook = 確実に縛る / Subagent = 文脈を分ける**。4つを組み合わせるのが2026年の主流。
- 人気の中心は **GitHub・Slack MCP**（汎用度が高い）と、**Anthropic 公式 Office スキル**（資料作成の自動化）。
- 本サーバでまず試す価値が高いのは: ①Office スキル×既存 Excel/PPT MCP での資料自動化、②GitHub MCP での開発ループ、③Playwright MCP でのブラウザ業務。

**提案**: 興味の強い「資料作成自動化（Officeスキル）」か「開発ループ（GitHub MCP）」のどちらかを、小さな PoC（概念実証）として 1 本作ってみるのが入口として最適です。ご希望あればタスク化します。

---

## 出典（2026年時点・参考）
- Cubitrek「The 18 Best MCP Servers in 2026」: [https://cubitrek.com/blog/best-mcp-servers-2026](https://cubitrek.com/blog/best-mcp-servers-2026)
- PopularAiTools「50 Best MCP Servers 2026」: [https://popularaitools.ai/blog/50-best-mcp-servers-2026](https://popularaitools.ai/blog/50-best-mcp-servers-2026)
- VoltAgent「awesome-agent-skills」(1400+収録): [https://github.com/VoltAgent/awesome-agent-skills](https://github.com/VoltAgent/awesome-agent-skills)
- scriptbyai「10 Best Agent Skills for Claude Code 2026」: [https://www.scriptbyai.com/best-agent-skills/](https://www.scriptbyai.com/best-agent-skills/)
