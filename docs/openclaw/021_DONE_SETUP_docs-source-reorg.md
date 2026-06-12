# 021_DONE_SETUP_docs-source-reorg.md - ドキュメント/ソース整理・正本統一

> STATUS: DONE / CATEGORY: SETUP / 作成日: 2026-06-12
> docs の3コピー（`/opt/docs` マスター・workspace チェックアウト・GitHub 公開リポ）の乖離を解消し、
> **GitHub を正本（source of truth）**として統一。併せて公開リポのマスキング違反1件を是正、
> タスクボードのフロントに残っていた旧状態デッドコードを除去した。デグレード無し。

## 背景（乖離の実態）

- `/opt/docs/openclaw`（宣言上のマスター）が GitHub より遅れ、**016/018/019 が欠落**、**015 が旧名（TODO/PLAN）**、
  **017 が番号衝突**（ローカル限定の `private-repo-backup` が公開済み `fable5-model-enable` と同番号）、規約外の実験残骸 `mcp-architecture-test.*` が混在。
- workspace の `013/014` が**マスキング前の古いドラフト**で、しかも git index に `A` でステージ済み（CLI commit で公開版を退行させ得る地雷）。
- 公開リポ未反映の研究doc `cfn-to-diagram-tool-selection.md` がローカルのみに存在。

## 実施内容

### 1. `/opt/docs/openclaw` マスターを GitHub 正本へ同期
- GitHub から `015_DONE_PLAN_poller-cost-redesign` / `016` / `017_DONE_SETUP_fable5-model-enable` / `018` / `019` を取得して反映。
- 旧名 `015_TODO_PLAN_taskboard-poller-cost-redesign.md` と実験残骸 `mcp-architecture-test.md/.mmd` は `_trash/<timestamp>/` へ退避（物理削除しない）。
- 番号衝突の `017_DONE_SETUP_private-repo-backup.md`（ローカル限定）を **`020_DONE_SETUP_private-repo-backup.md` へリネーム**（見出しも更新）。GitHub には push しない。
- 結果、マスターは `001`〜`020` の連番で整合。

### 2. 公開リポのマスキング違反を是正（重要）
- `015_DONE_PLAN_poller-cost-redesign.md` にポーラの**実ジョブID**が残存し、GitHub 公開版にもそのまま載っていた。
  `<poller-job-id>` へマスクし `/opt` と GitHub 両方を修正（016 と整合）。
- 注: 過去コミットには旧値が残るが、資格情報ではない内部 UUID のため履歴改変（force-push）は行わない。

### 3. workspace の整理
- workspace の `docs/openclaw/013〜019` をマスター（=GitHub）の正版で上書き同期。`020` は非公開のため workspace へは置かない。
- git index にステージされていた古いドラフト（`013` / 旧 `windows-update-summary`）を `git rm --cached` で index から除去（作業ファイルは保持）。
  ※当リポジトリはコミット0件（HEAD 未誕生）で、公開は github-mcp API 経由。index は実質未使用のため defuse 目的。

### 4. 未反映doc の push
- `docs/info/research/20260610_INFO_RESEARCH_cfn-to-diagram-tool-selection.md` をマスキング・機密スキャン後に GitHub へ追加。

### 5. ソース（tasks/task-board）の軽微なデッドコード除去
- フロントの旧状態（`processing` / `reviewing`、016 で廃止済み）の残骸を除去:
  - `public/index.html`: `.s-processing` / `.s-reviewing` CSS 規則を削除。
  - `public/home.html`: サマリ並び順配列から `processing` / `reviewing` を削除（`queued_exec` は実行待ち表示用エイリアスのため保持）。
- ソースは既に層構造（`src/domain｜application｜infra｜interface｜repository`）へリファクタ済みで、全ファイル `node --check` OK。
  `db.mjs` 等の `intake/processing/reviewing` 参照は**起動時マイグレーション・後方互換コード**でありデッドではない（保持）。

## 検証
- マスキング全スイープ（`/opt/docs/openclaw` ＋ workspace docs ＋ `docs/info`）で実ホスト名/IP/実ユーザ名/Discord ID/実ジョブID の漏れ **0件**。
- フロント編集後に旧状態トークンの残存なし、`node --check` 全ソース OK、保持すべき状態クラスは無傷。
- workspace git index ステージ **0件**。
- `/opt` マスターは `001`〜`020` 連番で整合。

## 運用メモ
- 以後 docs の正本は **GitHub `public-openclaw-01`**。`/opt/docs` は同一内容を保つマスター、workspace は公開セットのチェックアウト（`020` 等のローカル限定docは含めない）。
- ローカル限定doc（例: `020_private-repo-backup`、`ad-replace/`）は GitHub へ push しない。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
