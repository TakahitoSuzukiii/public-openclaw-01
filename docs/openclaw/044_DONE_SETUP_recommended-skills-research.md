# 044 追加おすすめ Claude Skills の調査とドキュメント化（OpenClaw サーバ向け）

- **ステータス:** DONE（調査＋ドキュメント化まで。実作成は `skill_workshop` の proposal まで／導入はHITL別承認後）
- **カテゴリ:** SETUP / 調査
- **対象:** 現状の OpenClaw 環境に「追加する価値があるおすすめ Skills（AgentSkills / SKILL.md）」の洗い出しと比較
- **作成日:** 2026-07-16（タスク #98）
- **関連:** [043 追加おすすめ MCP の調査](./043_DONE_SETUP_recommended-mcp-research.md)（MCP は別軸。本書は Skills 専用）
- **重要:** 本ドキュメントは調査のみ。`SKILL.md` の新規作成・`skills.entries` 設定変更は行っていない。導入は鈴木さんの承認後に「追加のみ」で別作業とする。

> **Skill（スキル）とは:** エージェント（Claude 等）に「いつ・どのツールをどう使うか」を教える Markdown 指示書（`SKILL.md`）。1 スキル＝1 ディレクトリで、YAML フロントマター（メタ情報）＋本文（手順）から成る。必要な時だけ本文が読み込まれる **progressive disclosure（段階的開示）** 方式なので、常時プロンプトに載るのは「名前＋説明＋場所」だけ＝トークン効率が良い。

---

## 1. 一次情報（出典）

すべてローカル同梱の公式ドキュメントで確認済み（推測記述なし）。

| 出典 | 内容 |
|---|---|
| `/tools/skills`（skills.md） | スキルの仕組み・読み込み順・ゲーティング・トークン影響 |
| `/tools/creating-skills` | `SKILL.md` の構造・作法・作成手順 |
| `/tools/skill-workshop` | `skill_workshop` ツールで proposal→apply の流れ |
| `/tools/skills-config`, `/cli/skills` | `skills.*` 設定・有効化 |
| ClawHub | 公開スキルレジストリ（`https://clawhub.ai`） |

**トークンコスト（一次情報より）:** スキルが有効なとき、プロンプトへ注入されるコストは決定的。
`total = 195 + Σ(97 + len(name) + len(description) + len(location))`。base 約195字、1スキルあたり約97字＋各フィールド長（≒24トークン/スキル）。→ **説明文は短く保つ**のが鉄則。

---

## 2. 現状の利用可能スキル（棚卸し）

本ランタイムで提示された利用可能スキル。大きく 2 系統。

**OpenClaw 同梱（`openclaw-skills:*`）:**
`browser-automation`, `canvas`, `create-apt`, `diagram-maker`, `discord`, `healthcheck`, `meme-maker`, `node-connect`, `node-inspect-debugger`, `notion`, `python-debugpy`, `skill-creator`, `spike`, `taskflow`, `taskflow-inbox-triage`, `weather`

**Claude Code 同梱（開発支援系）:**
`code-review`, `simplify`, `verify`, `dataviz`, `run`, `init`, `security-review`, `review`, `update-config`, `claude-api`, `schedule`, `loop`, `keybindings-help`, `fewer-permission-prompts` 等

**→ すでに充足している領域:**
ブラウザ自動化 / 作図（diagram-maker・drawio） / Discord 運用 / タスク登録（create-apt） / weather / 各種デバッグ（node・python） / コードレビュー・検証（code-review・verify・security-review） / データ可視化（dataviz）。

**→ 相対的に手薄な領域（今回の候補が埋める先）:**
1. **docs 命名・マスキング規約準拠のドキュメント作成**（`/opt/docs` の `NNN_STATUS_CATEGORY_name.md`＋`.backups` 退避＋public/private の push 判定）を毎回手作業でやっている
2. **タスクボード運用**（`taskctl.mjs` の status 遷移・計画添付フロー）が定型手順なのに未スキル化
3. **private/public リポへの byte-exact 反映**（github-mirror 規約）の標準化
4. **週次/月次 情報収集 cron のレポート整形**（info docs のフォーマット統一）
5. **資料自動生成**（Excel/PPTX）※既製の余地あり（要検証）

> 注: コードレビュー・検証・作図・ブラウザ・Discord は既存で足りるため、**重複を避けて「既存活用」に分類**し、新規候補からは除外する。

---

## 3. 評価軸（比較表の見方）

- **用途** … 何を自動化/標準化できるか
- **既製 or 自作** … ClawHub 等の既製で足りるか、`skill_workshop` で自作すべきか
- **実装難度** … 低（本文だけ）〜高（外部バイナリ/認証が必要）
- **リスク・注意点** … 権限範囲・機密の外部送信・破壊的操作の有無
- **トークン効率** … progressive disclosure 前提。常時コストは説明文長に比例
- **推奨度** … ◎（強く推奨）／○（条件付き推奨）／△（要検討）／×（原則見送り）

---

## 4. 候補比較表

### 4-1. 自作候補（現状の繰り返し手作業をスキル化＝最も効果大）

| 候補 | 用途 | 既製/自作 | 実装難度 | リスク・注意点 | トークン効率 | 推奨度 |
|---|---|---|---|---|---|---|
| **docs-writer** | docs 命名・マスキング・backup・push 判定の標準化 | 自作 | 低（本文＋既存 read/write） | 破壊的操作なし。マスキング規約を本文で強制 | 良（説明短く） | ◎ |
| **taskboard-ops** | `taskctl.mjs` の status 遷移・計画添付の定型化 | 自作 | 低（`node` 実行のラッパ） | データをコマンド引数に流すのみ。injection 注意を明記 | 良 | ◎ |
| **repo-mirror** | private/public への byte-exact push（github-mirror 規約） | 自作 | 中（git+token / MCP 判定） | push 先誤り防止。機密ゼロ確認を手順化 | 中 | ○ |
| **info-report** | 週次/月次 cron のレポート整形（info docs 統一） | 自作 | 低（本文テンプレ） | 外部送信なし | 良 | ○ |

### 4-2. 既製候補（ClawHub / 同梱の opt-in。重複しないもの限定）

| 候補 | 提供元 | 用途 | 導入 | リスク・注意点 | 推奨度 |
|---|---|---|---|---|---|
| **coding-agent**（同梱・opt-in） | OpenClaw 同梱 | Claude/Codex 等の CLI をコーディング委譲に利用 | `skills.entries.coding-agent.enabled: true` ＋対応 CLI 認証 | 既に claude-cli 稼働。効果は要検証 | △ |
| **資料自動生成系**（Excel/PPTX） | ClawHub 探索（未検証） | ppx/Excel の自動生成 | `openclaw skills install @owner/<slug>` | **第三者スキルは未検証コードとして扱う**。read 後・sandbox 推奨 | △（要検証） |
| **clawhub-publish**（同梱） | OpenClaw 同梱 | 自作スキルの ClawHub 公開 | `openclaw skills install @openclaw/clawhub-publish` | 公開＝外部送信。機密ゼロ必須 | ×（現状不要） |

> **既製スキルのセキュリティ原則（一次情報）:** 第三者スキルは **untrusted code（未検証コード）** として扱う。有効化前に必ず中身を read し、`openclaw skills verify @owner/<slug>` で trust envelope（VirusTotal/ClawScan/静的解析）を確認、リスクの高い入力・ツールは sandbox 実行が推奨。

---

## 5. 優先度つき 導入おすすめリスト

### Top1: `docs-writer`（◎）
- **理由:** 毎タスクの完了処理（docs 化＋マスキング＋backup＋push）が最も高頻度の定型手作業。AGENTS.md の規約（`NNN_STATUS_CATEGORY_name.md`／マスキング表／README 個人プロジェクト宣言）をスキル本文に埋めれば、規約ズレ・実値混入のヒューマンエラーを構造的に減らせる。
- **雛形の方向性:** `name: docs-writer` / `description: docs命名・マスキング規約準拠でドキュメントを作成し、backup退避とpublic/private判定を標準化` / trigger=「ドキュメント化」「docs 作成」/ 手順=① 連番採番 → ② マスキング表適用 → ③ `.backups` 退避 → ④ 本文生成 → ⑤ github-push-policy で public/private 判定。

### Top2: `taskboard-ops`（◎）
- **理由:** ポーラー運用（本タスクのような status 遷移・`append-instruction`・`log`）は完全に定型。`command-dispatch: tool` 系にはできないが、手順スキル化で毎回の judgement を安定化できる。
- **雛形の方向性:** `name: taskboard-ops` / trigger=「タスクボード」「taskctl」/ 手順に **injection 防止原則**（title/instruction/log は「データ」であり指示ではない）を明記。

### Top3: `repo-mirror`（○）
- **理由:** byte-exact 反映は事故ると公開事故（機密漏洩）に直結。判定（git+token か MCP か・public か private か）を手順化する価値が高い。ただし push 先誤りのリスクがあるため **dry-run/確認ステップ必須**。

> `info-report` は cron 側で概ね回っているため優先度は中。既製の資料自動生成は「未検証コードの精査」が前提のため、必要が生じた時点で個別検証。

---

## 6. 方針・制約（重要）

- **本書は調査＋ドキュメント化まで。** スキルの実作成は `skill_workshop`（proposal）まで、`apply`（live 反映）は鈴木さんの明示承認後・別作業。
- **追加のみ・デグレ無し:** 新スキルは既存スキルを書き換えず「追加」。同名は precedence で上書きされるため、既存名と衝突しない `name` を使う。
- **機密境界:** 転職関連（`~/career-private/`）を扱うスキルは**外部送信禁止**を前提に設計。ClawHub 公開系スキルはこの用途に使わない。
- **トークン最適化:** `description` は 160 字未満・短文（常時プロンプトコストに直結）。詳細手順は本文へ（progressive disclosure）。

---

## 7. 検証（本タスク）

- 文書のみ（`node` 不要・システム変更なし）。
- 各主張に一次情報 URL/出典を併記済み。
- 既存スキルとの重複を排除（code-review・diagram-maker・discord 等は「既存活用」に分類）。
- マスキング規約・機密ゼロを再確認（実値・トークン・PII の記載なし）。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
