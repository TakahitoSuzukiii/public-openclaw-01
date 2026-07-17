# 046_DONE_SETUP_infohub-topic-categories.md - 情報ハブのカテゴリ細分化（TOPICコード分類）

> 関連: `033_DONE_SETUP_taskboard-reports-and-info-hub.md`（情報ハブ初版）。作成日: 2026-07-17。
> 方針: 追加のみ＝デグレ無し・最小変更（フロント `public/info.html` のみ）。

## 1. 目的

`/info` の記事一覧で、システム開発・設計・アーキテクチャ・セキュリティ等の「技術観点」とは別の観点（ヘルスケア／金融／機械学習／自己啓発／その他）を**別カテゴリ**として整理する。あわせて各カテゴリ内は**最新記事が上**に来るようソートする。

## 2. 設計（カテゴリ分類ルール）

- `docs/info/news/` `docs/info/security/` `docs/openclaw/` は従来どおり**パス接頭辞**で分類。
- `docs/info/research/` 配下は、ファイル名の **TOPIC コード**（`YYYYMMDD_INFO_<TOPIC>_...`）で観点別に細分化する。

| カテゴリ key | 表示 | 対象 TOPIC コード |
|---|---|---|
| techresearch | 🔬 技術リサーチ | 上記以外の全 TOPIC（RESEARCH/ARCH/AWS/AWSAUTO/CLAUDEAUTO/AUTODOC/MCP/PLAYWRIGHT/SRE/AI/AIGOV/TEMPLATES/RESOURCES 等・**未登録TOPICの既定**） |
| ml | 🤖 機械学習 | ML |
| health | 🩺 ヘルスケア | HEALTH |
| finance | 💰 金融 | FINTECH, MONEY, FINANCE |
| selfdev | 📚 自己啓発 | CLASSICS, SELFDEV |
| other | 🗂 その他 | LIFE, MISC, OTHER |

- **ソート**: 細分カテゴリはフラットな**ファイル名降順（＝日付降順・最新が上）**。news/security/openclaw はサブディレクトリ見出しを維持し、各群内で最新が上。

## 3. 実装（`public/info.html` のみ）

- `CATS` を9カテゴリへ拡張（key/emoji/label/ja/desc、grouped フラグ）。
- 追加関数：`topicOf(path)`（TOPIC 抽出）／`TOPIC_CAT`（TOPIC→カテゴリ表）／`categorize(path)`（記事→カテゴリ key）。
- `showHub` の件数集計、`showCategory` のフィルタ/描画、`catKeyForPath`（記事本文の戻り先）を `categorize()` ベースへ変更。ルーティング/API/バックエンドは無変更。

## 4. 今後の運用ルール（恒久）

- 新しい調査ドキュメントは `docs/info/research/` に置き、**TOPIC コードで観点を表す**だけで自動的に正しいカテゴリへ入る。
- 非技術の観点は上表の TOPIC を使う（例: 健康=HEALTH、金融=FINTECH、機械学習=ML、古典/思想=CLASSICS、生活/グルメ=LIFE）。技術観点は任意の TOPIC（既定で技術リサーチに入る）。

## 5. 検証

- `public/info.html` の inline JS を抽出し `node --check` → OK。
- ライブ回帰（`http://127.0.0.1:<taskboard-port>`）：`/info` `/` `/dashboard` `/api/home` `/api/info/tree` `/api/reports` すべて 200。
- 分類シミュレーション（実ツリーに `categorize()` 相当を適用）：health=タンパク質／finance=X Money／ml=機械学習3件／selfdev=古典3件／other=居酒屋、技術系は techresearch、news/security/openclaw 温存。最新が上を確認。
- ブラウザ（Playwright headless）で `/info` ハブ（9カテゴリ＋レポート・件数一致）と `#cat=health`（対象記事表示）を確認。
- 着手前バックアップ：`~/.openclaw/workspace/.backups/task-infopage-categories-<ts>/info.html`。

## 6. 完了処理

- 構築ドキュメント（`/opt/docs/openclaw/`）＋公開リポジトリ `docs/openclaw/` へミラー（マスキング規約遵守）。
- 恒久ルールを NEXUS メモリへ登録。
