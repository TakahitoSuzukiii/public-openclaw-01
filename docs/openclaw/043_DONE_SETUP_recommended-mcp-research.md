# 043 追加おすすめ MCP の調査とドキュメント化（OpenClaw サーバ向け）

- **ステータス:** DONE（調査＋ドキュメント化まで。実導入は別途 HITL 承認後）
- **カテゴリ:** SETUP / 調査
- **対象:** 現状の OpenClaw サーバに「追加する価値があるおすすめ MCP」の洗い出しと比較
- **作成日:** 2026-07-09（タスク #97）
- **重要:** 本ドキュメントは調査のみ。config（`mcp.servers`）変更は行っていない。導入は鈴木さんの承認後に「追加のみ」で別作業とする。

> **MCP とは:** Model Context Protocol（モデル・コンテキスト・プロトコル）。AI エージェント（Claude 等）が外部ツールやデータへ安全にアクセスするための共通規格。MCP サーバを 1 つ追加すると、その分だけエージェントが使える道具（ツール）が増える。

---

## 1. 現状の導入済み MCP（棚卸し）

`openclaw mcp list` の実機確認結果。OpenClaw 管理下（`~/.openclaw/openclaw.json` の `mcp.servers`）は **4 つ**。

| 名前 | 役割 | カバー領域 |
|---|---|---|
| `github-mcp` | GitHub 操作（Issue/PR/リポジトリ/コード検索） | ソース・ドキュメント公開 |
| `aws-mcp` | AWS ドキュメント検索・リージョン情報等 | AWS/インフラ調査 |
| `playwright-mcp` | ブラウザ自動化（週次トレンド収集・E2E 確認） | Web 自動操作 |
| `drawio-mcp` | 図（アーキ図・フロー図）の作成 | ドキュメント作図 |

**→ すでに充足している領域:** GitHub 操作 / AWS ドキュメント / ブラウザ自動化 / 作図。

**→ 相対的に手薄な領域（今回の候補が埋める先）:**
1. 永続メモリ／ナレッジグラフ（セッションをまたいだ知識保持）
2. スコープ限定のローカルファイル横断操作
3. ローカル Git リポジトリの読み書き（GitHub API を介さない）
4. 最新ライブラリ/フレームワークの正確なドキュメント注入（ハルシネーション対策）
5. 意味検索ベースのコード編集支援

> 注: `notion` は skill として既にあり、SaaS 連携 MCP は機密境界の観点から原則見送り（後述）。

---

## 2. 評価軸（比較表の見方）

各候補を次の 7 軸で評価する。

- **用途 / 想定ユースケース** … 何ができるか
- **導入難度** … 低（`npx`/`uvx` で即）〜高（外部サービス設定が必要）
- **必要な認証・権限** … トークン・API キー・ファイルアクセス範囲
- **リスク** … 権限範囲・機密の外部送信の有無
- **トークンコスト** … ツール定義の重さ（多数のツールを持つ MCP は毎回のプロンプトに載り、コスト増）
- **メンテ状況** … 公式維持か・更新頻度
- **推奨度** … ◎（強く推奨）／○（条件付き推奨）／△（要検討）／×（原則見送り）

---

## 3. 候補比較表

| 候補 | 提供元 | 用途 | 導入難度 | 認証/権限 | リスク | トークンコスト | 推奨度 |
|---|---|---|---|---|---|---|---|
| **memory**（server-memory） | 公式 reference | ナレッジグラフ型の永続メモリ | 低（`npx`） | ローカルファイルのみ | 低（外部送信なし） | 小（ツール少数） | ◎ |
| **filesystem**（server-filesystem） | 公式 reference | パス限定のファイル操作 | 低（`npx`） | 指定ディレクトリのみ | 中（許可パス次第） | 中 | ○ |
| **git**（server-git） | 公式 reference | ローカル Git 読み書き・検索 | 低（`uvx`） | 対象リポのローカルパス | 低〜中（書込あり） | 中 | ○ |
| **fetch**（server-fetch） | 公式 reference | Web 取得＋Markdown 変換 | 低（`uvx`） | なし | 低（SSRF 注意） | 小 | △ |
| **context7** | Upstash（著名 OSS） | 最新ライブラリ公式ドキュメント注入 | 低（`npx`／HTTP） | 任意（APIキーで上限緩和） | 低（外部API問い合わせ） | 中 | ○ |
| **serena** | oraios（著名 OSS） | 意味検索ベースのコード解析/編集 | 中（`uvx`＋LSP） | プロジェクトのローカルパス | 中（コード書込） | 大（多ツール） | △ |
| **sequential-thinking** | 公式 reference | 段階的推論の補助 | 低（`npx`） | なし | 低 | 小 | △ |
| **time** | 公式 reference | タイムゾーン変換 | 低（`uvx`） | なし | 低 | 小 | △ |
| Slack / Notion / DB(SaaS) 等 | 各社 | 外部 SaaS 連携 | 中〜高 | 各種トークン | **高（機密外部送信）** | 大 | ×（原則見送り） |

---

## 4. 優先度つき 導入おすすめリスト

### ◎ Top 1: memory（server-memory）— 最優先

- **何が良いか:** OpenClaw には既にファイルベースのメモリ（`MEMORY.md`／日次ノート）があるが、これは **ナレッジグラフ**（エンティティと関係で知識を構造化）として保持する仕組み。人物・プロジェクト・依存関係のような「つながり」を扱う調査系タスクで効く。
- **セキュリティ:** ローカル JSON に保存、外部送信なし。低リスク。
- **トークン効率:** ツール数が少なく軽い。
- **一次情報:** https://github.com/modelcontextprotocol/servers/tree/main/src/memory
- **判断:** OpenClaw 既存メモリと役割が重なる面はあるので、**「既存で足りるか」を試用して見極め**てから。まず入れて損はない低リスク枠。

### ○ Top 2: context7 — ハルシネーション対策として実利大

- **何が良いか:** 「そのライブラリの最新の正しい使い方」を公式ドキュメントから引いてプロンプトに注入。エンジニア支援・コード生成で、古い/存在しない API を書く事故を減らす。
- **セキュリティ:** クエリを Context7 の API に送る（コード全体は送らない）。機密コードを含めない運用なら低リスク。APIキー無しでも利用可、キーでレート上限が緩和。
- **トークン効率:** 取得ドキュメントが本文に載るため、使う時だけ呼ぶ運用が前提（常時ロードは重い）。
- **一次情報:** https://github.com/upstash/context7
- **判断:** 鈴木さんのエンジニア支援文脈に直接効く。**○ で推奨。**

### ○ Top 3: git（server-git）— ローカル Git 操作

- **何が良いか:** GitHub API（github-mcp）を介さず、**ローカルの作業ツリー**で diff/log/blame/status を直接読める。タスクボード開発や `/opt/docs` 運用と親和。
- **セキュリティ:** 書込（commit 等）ツールを含むため、**読み取り主体で使う／対象パスを限定**する運用が安全。
- **トークン効率:** 中。
- **一次情報:** https://github.com/modelcontextprotocol/servers/tree/main/src/git
- **判断:** 現状は Bash 経由の `git` で足りている面もある。**必要性を感じたら追加**（○）。

### ○ 条件付き: filesystem（server-filesystem）

- ワークスペース外の特定ディレクトリ（例: `/opt/docs`）を横断操作したい時に有効。ただし **許可パスを厳格に絞る**こと。ワークスペースは既存ツールでカバーできるため、スコープが明確な時のみ（○）。
- 一次情報: https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem

### △ 検討: serena / fetch / sequential-thinking / time

- **serena:** 大規模コードベースの意味検索・リファクタに強力だが、ツール数が多く**トークンコスト大**、LSP セットアップも要る。大きめの開発タスクが定常化したら再検討（△）。一次情報: https://github.com/oraios/serena
- **fetch:** OpenClaw に既に `web_fetch` があり**機能が重複**。SSRF（内部アドレスへの到達）注意もあり、原則は既存で足りる（△）。一次情報: https://github.com/modelcontextprotocol/servers/tree/main/src/fetch
- **sequential-thinking / time:** 便利だが効果が限定的。前者はモデル自身の推論で代替可、後者は `session_status` で現在時刻が取れる（△）。

### × 原則見送り: SaaS 連携（Slack/Notion/外部 DB 等）

- 転職支援などの**ローカル機密**を扱う本サーバでは、機密を外部 SaaS に送りうる MCP は**機密境界の観点で原則採用しない**。どうしても必要な場合は、機密を含まないスコープに限定し、個別に HITL 承認。

---

## 5. 導入手順（骨子のみ・実行はしない）

> 実際の追加は鈴木さんの承認後に「追加のみ（`mcp.servers` へ 1 エントリ追記）」で別作業とする。デグレ防止のため既存 4 エントリは変更しない。

一般形（OpenClaw 管理下に追加する場合）:

```bash
# 例: memory サーバを追加（npx 実行型）
openclaw mcp add <名前> -- npx -y @modelcontextprotocol/server-memory

# 追加後の確認
openclaw mcp list
openclaw mcp tools <名前>   # 提供ツールとトークン負荷の確認
```

- Python 製（git/fetch/time）は `uvx <パッケージ>` 実行型になる。
- 追加後の**検証観点:** ①`mcp list` に出るか ②`mcp tools` でツール定義の重さを確認 ③想定外の外部送信・過剰権限がないか ④既存ツールとの重複で混乱しないか。
- 一次情報（コマンド仕様）: ローカル docs `/cli/mcp`（`openclaw mcp list|add|set|configure|tools|login`）。

---

## 6. リスク・前提・検証

- **本タスクのリスク:** 文書のみ → システムへの影響なし。
- **将来の実導入リスク:** `mcp.servers` への追加は「追加のみ」だが、（a）過剰権限、（b）機密の外部送信、（c）ツール定義肥大によるトークンコスト増 の 3 点を各候補で必ず評価してから採用する。
- **前提:** 導入可否の最終判断は鈴木さん（HITL）。転職関連の機密はローカル限定を厳守。
- **検証（本ドキュメント）:** ①現状 MCP を実機（`openclaw mcp list`）で確認済み ②各候補に一次情報 URL を併記 ③機密・トークンの記載ゼロ ④既存 4 MCP との重複を明示。

---

## 7. 一次情報（出典）

- 公式サーバ集: https://github.com/modelcontextprotocol/servers
- 公式サンプル一覧: https://modelcontextprotocol.io/examples
- memory: https://github.com/modelcontextprotocol/servers/tree/main/src/memory
- filesystem: https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem
- git: https://github.com/modelcontextprotocol/servers/tree/main/src/git
- fetch: https://github.com/modelcontextprotocol/servers/tree/main/src/fetch
- sequential-thinking: https://github.com/modelcontextprotocol/servers/tree/main/src/sequentialthinking
- time: https://github.com/modelcontextprotocol/servers/tree/main/src/time
- Context7（Upstash）: https://github.com/upstash/context7
- Serena（oraios）: https://github.com/oraios/serena
- OpenClaw MCP CLI: ローカル docs `/cli/mcp`

> 注: 公式 reference リポジトリは現在アクティブ維持が 7 種（everything / fetch / filesystem / git / memory / sequentialthinking / time）に整理され、旧来の一部サーバ（postgres, slack 等）はアーカイブ/別リポへ移動している。導入前に各リポの最新状態を確認すること。

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
