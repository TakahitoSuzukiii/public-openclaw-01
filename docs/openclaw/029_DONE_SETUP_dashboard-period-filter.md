# 029_DONE_SETUP_dashboard-period-filter.md - トレーニング/ボディメイク ダッシュボードの期間フィルタ

> 関連: `024_DONE_SETUP_training-dashboard.md`（トレーニングダッシュボード）/ `026_DONE_SETUP_taskboard-bodymake.md`・`027_DONE_SETUP_taskboard-body-composition-ocr.md`（ボディメイク）。
> 対象タスク: タスクボード #78（一般機能）。方針: **追加のみ＝デグレ無し**（既存テーブル・既存フロー非改変）。作成日: 2026-06-20。

## 1. 概要・目的

トレーニング（`/training/dashboard`）とボディメイク（`/body`）の各ダッシュボードに、**表示期間セレクタ**（直近1か月／3か月／6か月／1年）を追加し、集計・系列をその期間に絞り込めるようにする。

- 既存の `/api/training/stats`・`/api/body/stats` に **任意の `days` クエリを追加**するだけの後方互換拡張。`days` 未指定時は**従来どおり**の挙動。
- DB スキーマ・記録フローは一切変更しない（**集計の窓を変えるだけ**）。

## 2. 実装（追加・配線）

| 層 | ファイル | 役割 |
|---|---|---|
| application | `src/application/trainingService.mjs` | `dashboard(days)` に**期間引数**を追加。`days` 指定時は `今日-(days-1)` 以降に絞って集計＋系列を返す。`rangeDays` を応答に含める。`last7`/`last30` は従来どおり併記。 |
| application | `src/application/bodyService.mjs` | `dashboard(days)` に**期間引数**を追加。期間サマリ（開始値・通算増減・最小最大）と体重系列を期間に限定（未指定時は系列は直近30日）。 |
| repository | `src/repository/trainingRepository.mjs` | 集計に必要な範囲取得の微修正（既存メソッドの後方互換調整）。 |
| interface | `src/interface/httpRouter.mjs` | `GET /api/training/stats` `GET /api/body/stats` で `?days=` を読み取り、正の有限数のときのみ `dashboard(days)` に渡す（不正値は無視＝従来挙動）。 |
| UI | `public/training-dashboard.html` `public/body.html` | 期間セグメントUI（`#period-seg`）と `setPeriod(days)` を追加。既定 30 日で初期表示。期間ラベル（`直近1か月/3か月/6か月/1年`）を表示。 |

### 仕様メモ

- **期間プリセット**: `30 / 90 / 180 / 365`（日）。ラベルは `直近1か月 / 3か月 / 6か月 / 1年`。それ以外の日数は `直近N日` 表示。
- **境界**: `since = 今日 -(days-1)`（**本日を含む**窓）。`days` が非有限・0以下なら `undefined` 扱いで全期間/従来集計。
- **後方互換**: `days` 省略時の応答は従来と同一。`rangeDays`（=適用期間, null=全期間）を追加フィールドとして返すのみ。

## 3. 検証

- `node --check` 全対象ファイル → OK。サービス active。
- `GET /api/training/stats?days=30` → `rangeDays:30`、`summary/last7/last30/series/categoryDist/topExercises/weekday` を返却。
- `GET /api/body/stats?days=90` → `profile/current/summary/rangeDays/series30` を返却。
- **後方互換**: `days` 無しの `GET /api/training/stats` `GET /api/body/stats` → 従来応答を維持。
- **回帰（デグレ確認）**: `/training/dashboard` `/body` `/api/training/sets` `/api/body/logs` `/api/home` `/dashboard` → 影響なし。

## 4. 完了処理

- **private バックアップ**: GitHub private リポ `private-openclaw-01`（master）へ反映。**remote blob SHA をローカル `git hash-object` と突合し byte-exact 一致を確認**。
- **ドキュメント化**: 本ファイル（マスター: `/opt/docs/openclaw/`）＋公開リポジトリへミラー。

## 5. セキュリティ・マスキング上の注意

- トレーニング/身体データは機微情報。本ドキュメントに実測値は記載しない。
- タスクボードは loopback / Tailscale 限定。期間フィルタは集計の窓を変えるのみで外部送信なし。
