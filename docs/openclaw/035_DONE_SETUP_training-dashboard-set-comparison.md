# 035_DONE_SETUP_training-dashboard-set-comparison.md - トレーニングダッシュボード セット別比較（折れ線）

> 関連: `034_DONE_SETUP_training-dashboard-part-detail.md`（部位別詳細）/ `024_DONE_SETUP_training-dashboard.md`（本体）。対象タスク: #81（種目別比較グラフの改修・本人指示）。作成日: 2026-06-22。
> 方針: 追加のみ＝デグレ無し。既存テーブル（`training_sets`）・記録フロー・`dashboard()` は非改変。**履歴データは書き換えない**。

## 1. 背景・課題

「部位別の詳細」の種目比較は**日次の総ボリューム折れ線**だった（034）。本人評価＝「総ボリューム比較はわかりづらく意味が薄い」。
要望＝「次回トレ時に、各セット（1set目＝何kg×何回…）を前回・過去と並べて見て、今日の目標を立てたい」。
生データ `training_sets` は **1行=1セット（重量×回数）** を保持しているため、追加実装のみで実現可能。

## 2. 仕様（本人確定）

- **X軸＝セット番号**（左=1set目→右へ）、**Y軸＝重量（kg/lb）**。種目ごとに**折れ線を重ねて**表示。
- 直近のトレ日（セッション）は個別の線：**直近 / 前回 / 前々回**（=直近3回）。
- **前々回より過去**は平均集約して本数を抑える：
  - **3か月以内** … 3回分を1本に平均（破線）
  - **3か月超** … 5回分を1本に平均（点線）
- 凡例で各線を見た目で区別（色＋線種：実線/破線/点線、日付範囲付き）。
- **各セットの回数（レップ数）**を**全ての線・全ての点**に `×回数` で注記（線色に合わせ、上下振り分け＋背景縁取りで可読性を確保）。点ツールチップにも「重量×回数」。
- 単位は種目内が単一なら kg/lb をそのまま、混在時は kg 換算。

## 3. 実装（層構造踏襲・追加のみ）

| 層 | ファイル | 役割 |
|---|---|---|
| application | `src/application/trainingService.mjs` | `setComparison(days)` を追加（`dashboard()` 非改変）。`buildLines()`：直近3回=個別、残りを 3か月境界で `avg3`(3回)/`avg5`(5回) に平均集約。`avgPoints()` で重量・回数を平均。kg/lb 変換 `toDispUnit()`。 |
| interface | `src/interface/httpRouter.mjs` | `GET /api/training/set-comparison?days=` を追加。 |
| UI | `public/training-dashboard.html` | `multiLineChart()`（X=セット/Y=重量の重ね折れ線・線種スタイル・直近の×回数注記・点ツールチップ）と `renderCmpCharts()`（凡例＝色＋線種＋日付範囲）を追加。部位別詳細の総ボリューム折れ線を比較グラフへ置換（部位サマリKPIは据置）。`CMP_PARTS` を `load()` で併取得。 |

- repository は**変更なし**（既存 `listSets()` を再利用）。`lineChart()`（旧単系列）は未使用で残置（デグレ無し）。

## 4. API レスポンス構造

```
{ rangeDays, indiv:3, groupRecent:3, groupOld:5,
  parts:[ { part, sets, exercises:[ {
    exercise, unit, grouped, maxSet, sessions,
    lines:[ { label, kind:'indiv'|'avg3'|'avg5', n?, dates:[...],
              points:[ {set, weight, reps} ] } ]  // 新しい→古い
  } ] } ] }
```
- `kind`：`indiv`=直近/前回/前々回、`avg3`=3か月以内3回平均、`avg5`=3か月超5回平均。
- 部位＝セット数降順、種目＝直近実施日降順。重量は表示単位（混在時 kg）。

## 5. UI スタイル

- 線種：個別=実線（太・色別）／avg3=破線／avg5=点線。描画は平均線→個別線の順（個別を前面、直近最前面）。全線の各点に `×回数` を線色で注記（線インデックスで上下振り分け＋`paint-order:stroke` の背景縁取り）。
- 凡例：色スウォッチ（線種反映）＋ラベル（直近/前回/前々回/平均(3回)/平均(5回)）＋日付（範囲）。
- Y軸はキリの良い目盛（`niceAxis()`）：**kg＝10kgごとに数値ラベル＋5kgの補助薄線**、lb＝20/25/50lb 基調＋半分の補助線。下限はデータ最小の少し下から。

## 6. 検証

- `node --check`（service/router）OK／`systemctl --user restart openclaw-taskboard.service`。
- 正常系（実データ）: ベンチプレス＝2セッション→直近/前回（個別・レップ保持）。
- グルーピング: テスト種目「TESTベンチ」に11セッション投入 → **直近/前回/前々回＋平均(3回:05/25〜06/05)＋平均(5回:01/05〜03/10)** の計5本、平均値・レップ平均を実機（ブラウザ）で確認 → **投入データは全削除し原状復帰**（78 set・TESTベンチ 0）。
- 異常系: `days=1`（記録の無い期間）→ 空表示。`days=abc`（不正）→ 全期間扱いでクラッシュ無し。
- 回帰: `/api/training/stats`（summary.sets=78 不変）・`/api/training/sets`・`/training/dashboard` 全 200。コンソール JS エラー無し（favicon 404 のみ・既存）。
- 備考: ヘッドレス・ブラウザのスクショは CJK グリフが一部欠落する（**スクショ限定の表示問題**で、実ブラウザ・DOM テキストは正常）。

## 7. セキュリティ・マスキング

- 機密情報なし。個人のトレーニング記録のみ。ホスト名/IP/実ユーザ名は本書に含めない（`127.0.0.1` のローカル参照のみ）。

## 8. 切り戻し

- 着手前バックアップ: `~/.openclaw/workspace/.backups/task81-set-comparison-20260622-140118/`（`trainingService.mjs`/`httpRouter.mjs`/`training-dashboard.html`/`tasks.db`）。
