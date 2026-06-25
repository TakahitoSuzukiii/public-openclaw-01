# 038 NEXUS 小改修3点: ファビコン / Claude CLI バージョン表示 / レポートのデモ公開

> タスクボード(NEXUS)の小改修3件の記録。いずれも「追加のみ＝デグレ無し」。層構造(interface→application→domain→repository→infra)を踏襲。
> マスキング規約により実値(IP/ホスト名/ユーザ名)は placeholder。

関連: [[037 Caddy IP制限デモ]]（#82/#83 のデモ公開基盤）。

---

## 1. ファビコン追加（全画面・#92）

- **目的**: 全画面にブランドアイコンを表示し見栄えを向上。
- **成果物**: `public/favicon.svg`（NEXUS のヘキサゴン型ノード・エンブレム。紫グラデの「メカ/AI秘書」風・ベクター）。
- **配線**:
  - 配信ルート: `GET /favicon.svg` と `GET /favicon.ico`（同SVGで応答し 404 抑止）を interface 層(httpRouter)に追加。
  - 各HTMLの `<head>` に `<link rel="icon" type="image/svg+xml" href="/favicon.svg" />` を追加（全ページ）。
- **ポイント**: SVG ファビコンはモダンブラウザ全般で表示可。`/favicon.ico` も同SVGを返すことで自動リクエストの 404 を防ぐ。

## 2. ホームに Claude CLI バージョン表示（#93）

- **目的**: ホーム「サーバ＆システム状況」に Claude CLI のバージョンを表示。
- **配線（層構造どおり）**:
  - infra: `getClaudeCliVersion()` を追加。`claude --version` を起動時に1回取得しキャッシュ、定期(30分)更新。失敗時は null 維持。実行ファイルは環境変数 `CLAUDE_BIN`（既定は標準インストールパス）で解決。
  - application: `systemStatus()` の `meta` に `claudeCliVersion` を追加。
  - interface(画面): ホームの `renderMeta` に「Claude CLI バージョン」の行を追加。
- **表示例**: `2.1.190 (Claude Code)`。
- **ポイント**: CLI 実行はキャッシュ前提（毎リクエストで叩かない）。systemd 配下では PATH が限定されるため**絶対パス(env 上書き可)**で解決する。

## 3. レポートをデモサイトにも公開（#90）

- **背景**: #82 のデモ公開ではレポートを遮断していた（「必要なら別途依頼」とされていた分）。
- **変更**: デモ経路でもレポートを閲覧/DL可能にする。**career/chat は引き続き遮断**（多層防御は維持）。
  - アプリ(interface): `X-Demo` ガードの遮断対象から reports を除外（career/chat は維持）。
  - リバースプロキシ(Caddy): `@blocked` から reports 系パスを除外。**設定反映には Caddy の再配置＋再起動が必要**。
- **⚠️ 公開ポリシー**: 公開してよいレポートは**公開済みデータと同系統(トレーニング/ヘルスケア系)に限る**。キャリア・個人メモ・機微情報を含むレポートは公開対象外。公開前に中身を確認する運用とする。
- **検証**: デモ経路で reports=200、career/chat=403、書込拒否・サンプルのみ表示は維持（デグレ無し）。

---

## 検証（共通）

- `node --check`（変更した .mjs）／タスクボード サービス再起動／主要エンドポイントのHTTPコード確認。
- 既存画面・API のデグレが無いこと（追加のみ）を確認。

## ロールバック

- 変更前バックアップ `.backups/tasks-*` から `src`/`public`/`deploy` を戻し、サービス再起動。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
