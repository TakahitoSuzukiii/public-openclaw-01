# 033_DONE_SETUP_taskboard-reports-and-info-hub.md - レポート画面＋ドキュメント目次ハブ

> 関連: `031_DONE_SETUP_weekly-doc-automation-cron.md`（週次レポート自動生成 cron）。対象タスク: タスクボード #80（追加対応）。作成日: 2026-06-21。
> 方針: 追加のみ＝デグレ無し。

## 1. 概要・目的

2点を追加する。

1. **レポート画面**: 自動生成した週次レポート（Excel 等）を**ブラウザから確認・ダウンロード**できるようにする。個人データを含むため**公開せず、ローカル/社内ネットワークからのみ**配信。
2. **ドキュメント目次ハブ**: `/info` を目次画面にし、配下に **News / Research / Security / OpenClaw / レポート** の各画面を設けて整理する。

## 2. 実装

### レポート画面（バックエンド：層構造で追加）

| 層 | ファイル | 役割 |
|---|---|---|
| infra | `src/infra/reports.mjs` | **新規**。`data/reports/`（gitignore 配下）の安全な読取。拡張子 whitelist（xlsx/pptx/pdf/docx/csv）・**パストラバーサル遮断**（単一ファイル名のみ許可）。 |
| application | `src/application/reportsService.mjs` | **新規**。ファイル名から**分かりやすい表示名・日付・種別・DL名**を導出（例: `週次トレーニングレポート`、DL名 `週次トレーニングレポート_2026-06-21.xlsx`）。 |
| interface | `src/interface/httpRouter.mjs` | **配線**。`GET /api/reports`（一覧）/ `GET /reports/dl?f=`（添付ダウンロード・UTF-8 日本語ファイル名）。 |

### ドキュメント目次ハブ（フロント）

- `public/info.html` を再構成。ハッシュルーティング：`#`（目次ハブ）/ `#cat=news|research|security|openclaw`（カテゴリ別記事一覧）/ `#cat=reports`（レポート画面）/ `#path=...`（記事本文）。
- カテゴリは GitHub `docs/` 配下のパス接頭辞で分類（news=`docs/info/news/`、research=`docs/info/research/`、security=`docs/info/security/`、openclaw=`docs/openclaw/`）。ハブに各カテゴリの件数を表示。

## 3. プライバシー設計（重要）

- 週次レポートは身体・トレーニング等の**個人データを含むため公開リポへ出さない**。`data/reports/` はローカル限定（gitignore）。
- レポート配信は loopback / Tailscale（tailnet 限定・Funnel 無効）からのみ。ダウンロードは常に `attachment`。

## 4. 検証

- `node --check` 全対象 OK／サービス再起動／`GET /api/reports` で一覧・親切な表示名/DL名を確認／`GET /reports/dl?f=` で 200・正しい content-type・UTF-8 日本語ファイル名・有効ファイルを確認。
- **パストラバーサル**：`../tasks.db`・`../../server.mjs` → 404（遮断）。
- 回帰: `/info` `/` `/api/home` `/dashboard` 他 全 200。ブラウザで目次ハブ・カテゴリ・レポート画面の表示/DL を確認。

## 5. 完了処理

- private バックアップ `private-openclaw-01`（master）へ反映・byte-exact 一致確認。
- 本ドキュメント（`/opt/docs/openclaw/`）＋公開リポジトリへミラー。
