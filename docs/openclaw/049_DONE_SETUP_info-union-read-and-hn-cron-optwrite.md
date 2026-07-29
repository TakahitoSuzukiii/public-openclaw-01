# 049 /info 表示漏れの恒久対策 — infoDocs 統合読み取り ＋ HN cron の /opt master 書き込み

- STATUS: DONE / CATEGORY: SETUP
- 対応日: 2026-07-29
- 対象: Nexus タスクボード `/info`（情報ハブ）と週次 Hacker News cron

## 背景・課題

`/info` は未認証の GitHub raw/API 直叩きによるレート制限(429)を避けるため、ローカルの `docs/` 配下を配信する（`infoDocs.mjs`）。しかし従来は **`workspace/docs` の1ルートのみ**を読んでいた。

一方、週次 cron 群（trending / al2023 / claude-auto / aws-auto / windows-update）は記事の**マスターを `/opt/docs/openclaw-news/info/` 配下（＋GitHub）にのみ書き込み**、workspace チェックアウトを更新しない。このため **cron 出力が `/info` に出ない**不具合が発生していた。

## 対策（2段階・いずれも「追加のみ・デグレ無し」）

### 第1弾: infoDocs.mjs を統合読み取り化

`src/infra/infoDocs.mjs`（タスクボード層構造の infra 層）を、以下2ルートの **union（和集合）配信**に改修した。

- ルート1: `workspace/docs`（従来どおり・優先）
- ルート2: `/opt/docs/openclaw-news`（cron 群がマスターを書く場所）

実装ポイント:
- `DOCS_ROOT` に加え `DOCS_ROOT_EXTRA`（既定 `/opt/docs/openclaw-news`）を追加。環境変数 `INFO_DOCS_ROOT` / `INFO_DOCS_ROOT_EXTRA` で上書き可能。
- `listDocs()` は全ルートを列挙し、**パス重複は先勝ち（workspace 優先）**で1件に集約。
- `readDoc()` は各ルートを順に探索。各ルートでパストラバーサル対策（解決後もルート配下であること）を維持。
- 効果: `/opt` マスターに書く5本の cron は **cron を一切変更せず**今後自動で `/info` に反映される。

### 第2弾: weekly-hackernews cron だけ /opt master にも書くよう修正

HN cron は他5本と異なり、記事を **ローカルに一切書かず GitHub push のみ**だったため、統合読み取りでも拾えなかった。そこで cron のプロンプト（`payload.message`）に手順「4-2」を追加した。

- 追加手順（要旨）: GitHub push（手順4）の直後に `mkdir -p /opt/docs/openclaw-news/info/news` を実行し、**push したものと完全に同一の本文**を `/opt/docs/openclaw-news/info/news/<YYYYMMDD>_INFO_HN_hackernews-weekly.md`（ファイル名は手順4と同一）へ書き込む。
- これで他5本と同じ「/opt master に書く」形に揃い、統合ビューアが自動表示する。

## 実施手順（cron 編集の安全対応）

cron 定義は SQLite（`state/openclaw.sqlite` の `cron_jobs` テーブル）が権威。**手編集は非推奨で、CLI / Gateway API を使う**のが公式方針（`docs/cli/cron.md`）。

既知のリスク: **mcp cron ツールでの編集は `toolsAllow` を再注入して壊す**恐れがある（過去の AMI 復元時の教訓）。そこで本対応では:

1. cron ストアをバックアップ（`openclaw.sqlite` + `-wal` + `-shm` を同時コピー）。保管: `~/.openclaw/workspace/.backups/openclaw.sqlite.<timestamp>.bak`。
2. **CLI 経路**で編集（mcp ツールではなく）:
   ```
   openclaw cron edit <job-id> --message "<新しい全文>"
   ```
   CLI は Gateway 経由のため in-memory スケジューラと DB を整合更新し、再起動不要。`--message` は指定フィールドのみ更新するため `toolsAllow` を保持する。
3. 検証（DB 直読み・読み取りのみ）:
   - `payload_tools_allow_json` = **null のまま**（＝全ツール許可を保持）。
   - `job_json.payload` に `toolsAllow` が**注入されていない**（keys = kind, message, model）。
   - `payload_model` / `schedule_expr` / `schedule_tz` / `session_target` / `delivery_*` / `next_run_at_ms` すべて不変。

## 検証結果

- `infoDocs.listDocs()` 実挙動で 2ルートを走査し `/opt` 側 news を配信することを確認（total 143 / news 18、直近の HN 記事も表示）。
- HN cron の CLI 編集後、上記 DB 検証項目すべて OK（`toolsAllow` トラップ回避を確認）。
- HN cron の次回実行は毎週水曜 06:10 JST（`10 6 * * 3`）。次回実行時に新手順が発火する。

## 切り戻し

- infoDocs.mjs: 着手前バックアップ `~/.openclaw/workspace/.backups/infoDocs.mjs.<timestamp>.bak`。
- cron ストア: 上記 `openclaw.sqlite.<timestamp>.bak`（+wal/+shm）で復元可能。

## 関連

- ソース（byte-exact）: private ソースミラー `src/infra/infoDocs.mjs`。
- `/info` カテゴリ運用: doc 046。HN cron 構築: doc 048。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
