# 036_DONE_SETUP_topbar-unify-and-healthcare.md - 全画面トップバー統一 ＆ ヘルスケア改名・主要指標グラフ

> 関連: `014_DONE_SETUP_task-board.md`（本体）。対象タスク: **#76（トップバー統一）** / **#77（ボディメイク管理→ヘルスケア・体組成グラフ）**。作成日: 2026-06-24。
> 方針: **追加のみ＝デグレ無し**。既存ロジックは最小差分でリファクタリング。層構造（interface → application → domain → repository → infra）踏襲。本人指示によりバックアップは省略。
> 公開範囲: 改修ソースは **private リポ（`private-openclaw-01`）** へ push。本ドキュメントはマスキング済みで `/opt/docs`（マスター）＋公開リポ（ミラー）。

## 1. 背景・課題

- **#76**: 全画面のトップバー（画面最上部のバー）が画面ごとに別実装で、構造・余白・ナビ表現がバラバラだった（旧 `index.html`・`info.html` はフラットなテキストリンク、他はピル型 nav など）。
- **#77**: 「ボディメイク管理」という画面名・機能名が分かりづらい。またサマリの体組成データが「現在値カード」中心で、過去推移がグラフで見えなかった。

## 2. 対応内容

### #76 トップバー統一（ベース＝ホーム画面）

- **共通CSS `public/topbar.css` を新設**。`header` / `.logo` / `h1` / `.nav` / `.navlink` / `.hright` / `.hbtn` の統一スタイルを集約（ホーム画面の意匠が基準）。更新ボタン `.hbtn` は各画面の汎用 `button` スタイルに依存しないよう自己完結のピル型に。
- **対象6画面**（ホーム/タスクボード/ドキュメント一覧(info)/トレーニングダッシュ/トレーニング記録/ヘルスケア）が `topbar.css` を読み込み、**各画面のインライン header CSS（重複定義）を削除**。マークアップも home パターン（ロゴ＝ホームリンク／`h1`＝画面名／`nav.navlink`／右側 `hright`）に統一。
- 配信: `httpRouter.mjs` に `GET /topbar.css`（`content-type: text/css`）を追加。
- ナビに「🩺 ヘルスケア」リンクを各画面へ追加（リンク先は `/body` のまま）。
- ※ キャリア配下（`/career*`）は対象外（別系統・独立ナビ）。

### #77 ヘルスケア改名 ＆ 主要指標グラフ

- **画面名・機能名を「ボディメイク管理」→「ヘルスケア」に変更**（タイトル・`h1`・全画面のナビ表記）。**URL `/body` と API `/api/body/*` は変更せず**（ブックマーク・OCR取込フロー互換のため。表示名のみ変更）。
- **サマリの体組成データを主要5指標の推移グラフ化**: 体重・体脂肪率・筋肉量・基礎代謝・体年齢。依存ゼロのインライン SVG をカード化し、2列グリッドで表示。体重カードは目標体重の水平線を併記。各指標は**記録のある日のみプロット**、2日分未満は「記録が2日分以上ありません」を表示。
- **application層 `bodyService.dashboard()`**: 日次系列 `series30` に `muscleMass` / `bmr` / `bodyAge` を追加（既存 `weight` / `bodyFat` に加算。**追加のみ・後方互換**）。各指標は同日複数記録を平均、基礎代謝・体年齢は整数丸め。
- データソースは既存の体組成カラム（`muscle_mass` / `bmr` / `body_age`、#27 体組成OCR拡張で導入済）を流用＝**スキーマ変更なし**。

## 3. 変更ファイル

| 層 | ファイル | 変更 |
|---|---|---|
| interface(UI) | `public/topbar.css` | **新規**。全画面共通トップバーCSS |
| interface | `src/interface/httpRouter.mjs` | `GET /topbar.css` ルート追加（配線のみ） |
| interface(UI) | `public/home.html` | topbar.css 参照・インラインheaderCSS削除（ベース）＋ナビ表記ヘルスケア化 |
| interface(UI) | `public/index.html` | topbar.css 参照・フラットheader→home型へ正規化（更新ボタン＋meta を hright に） |
| interface(UI) | `public/info.html` | topbar.css 参照・`a.nav`→`nav.navlink` 正規化 |
| interface(UI) | `public/training.html` / `public/training-dashboard.html` | topbar.css 参照・インラインheaderCSS削除・ナビ整理 |
| interface(UI) | `public/body.html` | topbar.css 参照・**ヘルスケア改名**・サマリを主要5指標グラフ化（`renderCharts()`/`metricSvg()`） |
| application | `src/application/bodyService.mjs` | `dashboard()` の日次系列に筋肉量/基礎代謝/体年齢を追加（追加のみ） |

- repository / domain / infra は**変更なし**（既存カラム・既存APIを再利用）。

## 4. 検証

- `node --check`（`bodyService.mjs` / `httpRouter.mjs`）OK。全変更HTMLのインラインJSを `vm.compileFunction` で構文検証 OK。
- `systemctl --user restart openclaw-taskboard.service` → active。
- ルート/API: `/` `/dashboard` `/info` `/training` `/training/dashboard` `/body` `/topbar.css` `/api/body/stats` すべて **200**。`/topbar.css` の `content-type: text/css` を確認。
- API回帰＋拡張: `/api/body/stats?days=365` の `series30` に `weight/bodyFat/muscleMass/bmr/bodyAge` が揃うことを確認（後方互換＝既存キー不変）。
- **ビジュアルリグレッション**（Chrome ヘッドレス・全描画完了後にキャプチャ）: 代表画面＝ヘルスケア（`/body`）。ヘッダ統一（共通topbar・ピルnav・`h1=🩺 ヘルスケア`）と主要5指標グラフ（体重＝目標線付き/体脂肪率/筋肉量/基礎代謝/体年齢）の描画を確認。
  - 検証用に体組成2日分を一時投入してグラフ描画を確認 → **投入データは削除し原状復帰**（記録3件に戻ることを確認）。
  - 備考: ヘッドレスのスクショは CJK グリフが一部欠落するが、DOM テキスト（`h1` 等）・実ブラウザ表示は正常（スクショ限定の既知事象）。
- デグレ確認: 既存の体重推移描画は主要指標グラフへ統合（旧 `renderChart`/`#chart` 参照は残存なし）。記録追加・削除・プロフィール保存・OCR取込導線は不変。

## 5. セキュリティ・マスキング

- 機密情報なし。UIテンプレート＋集計ロジックのみ（個人の体組成データは `/api/body/*`（loopback/tailnet）から実行時取得、コードに値は含まれない）。
- ホスト名/IP/実ユーザ名は本書に含めない（`127.0.0.1` のローカル参照のみ）。

## 6. 完了処理

- **private 反映**: `private-openclaw-01`(master) へ上記9ファイルを git+token で push。remote blob SHA をローカルと突合し **byte-exact 一致**を確認。
- **ドキュメント**: 本ファイル（マスター `/opt/docs/openclaw/`）＋公開リポ `public-openclaw-01`（`docs/openclaw/`）へミラー（マスキング厳守）。
