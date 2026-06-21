# 034_DONE_SETUP_training-dashboard-part-detail.md - トレーニングダッシュボード 部位別詳細＋腕分割

> 関連: `024_DONE_SETUP_training-dashboard.md`（ダッシュボード本体）/ `023_DONE_SETUP_training-management.md`（管理）。対象タスク: トレーニング管理の改修（本人指示）。作成日: 2026-06-21。
> 方針: 追加のみ＝デグレ無し。既存テーブル（training_sets）・記録フローは非改変。**履歴データは書き換えない**。

## 1. 変更点

1. **曜日別頻度を削除**（集計・画面パネルとも）。
2. **「腕」を「二頭筋」「三頭筋」に分割**（カテゴリ：胸/肩/二頭筋/三頭筋/背中/足/腹）。
3. **部位別の詳細**を新設：部位セレクタ → 部位サマリ＋種目ごとの推移グラフ（比較）。

## 2. 実装

| 層 | ファイル | 役割 |
|---|---|---|
| application | `src/application/trainingService.mjs` | `EXERCISES` の「腕」を二頭筋/三頭筋へ分割。種目名→部位の逆引き `EX_TO_CAT` ＋ `partOf()` を追加。`dashboard()` から `weekday` を除去し、**`parts`**（部位→種目→日次系列）を追加。 |
| UI | `public/training-dashboard.html` | 曜日パネル削除。`lineChart()`（SVG 折れ線）を追加。「🧩 部位別の詳細」パネル（部位チップ＋部位サマリ KPI＋種目ごとの折れ線グラフ）を追加。 |

## 3. 旧「腕」記録の扱い（非破壊リマップ）

- 既存記録は `category='腕'` で保存済み。**DB は書き換えず**、集計時に `partOf()`＝「種目名で逆引きした部位」を優先することで、旧「腕」記録（トライセプス系・ナローベンチプレス等）も**表示上は自動で三頭筋/二頭筋に振り分け**られる。
- 逆引きできない自由入力種目は保存カテゴリにフォールバック。

## 4. `parts` 構造（API 追加フィールド）

```
parts: [ { part, volKg, sets, reps, workouts,
           exercises: [ { exercise, volKg, sets, reps, maxWeightKg, est1RMKg,
                          series: [ { date, volKg, sets, reps, maxWeightKg } ] } ] } ]
```
- 部位・種目はボリューム降順、系列は日付昇順。重量は kg 換算（kg/lb 混在を横断比較可能に）。

## 5. 検証

- `node --check` OK／サービス再起動。
- `GET /api/training/stats?days=365`：`weekday` キーが無いこと、`parts` が部位→種目→系列を返すこと、旧「腕」が**三頭筋**に集約されることを確認。
- カテゴリ検証：`二頭筋` は受理、旧 `腕` は `unknown category` で拒否（投入テストデータは削除し原状復帰）。
- ブラウザ：部位チップ切替・部位サマリ・種目ごとの折れ線グラフ描画を確認。回帰 `/training/dashboard` `/training` `/api/training/sets` 他 全 200。

## 6. セキュリティ・マスキング

- 身体・トレーニングは機微データ。本ドキュメントに実測値は記載しない。タスクボードは loopback / Tailscale 限定。

## 7. 完了処理

- private バックアップ `private-openclaw-01`（master）へ反映・byte-exact 一致確認。
- 本ドキュメント（`/opt/docs/openclaw/`）＋公開リポジトリへミラー。
