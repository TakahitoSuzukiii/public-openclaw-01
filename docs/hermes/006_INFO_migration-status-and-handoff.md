# OpenClaw → Hermes 移行 現状サマリ & 引き継ぎ

> ステータス: INFO（進行中プロジェクトの現状記録）/ 更新 2026-08-09
> ⚠️ マスキング規約準拠。ユーザー名/ホスト名/ID/トークンは記載しない。
> 関連: `001`（Discord連携）／`002`（移行計画）／`003`（データ分類）／`004`（移行実行）／`005`（常駐化・ショートカット）

## 0. 目的と構成

既存 **OpenClaw**（クラウド/別ホスト）から、Windows 11 の **WSL 上 Hermes Agent**（エージェント名 **Optimus** 🔱）へ本格移行。OpenClaw は当面**並行稼働**のまま、Hermes 側を主運用に育てる。

## 1. 完了済み（2026-08-08〜09）

- **Discord 連携**: 新 bot・fail-closed 許可制・home channel 設定。
- **D ドライブ移設**: WSL ディストロの `ext4.vhdx` を D:（3TB）へ（`wsl --manage --move`）。SQLite(state.db) 含む全データが ext4 のまま D 上。
- **移行**: `hermes claw migrate --preset full --overwrite`（13件・Secrets 無し・新 bot 保護／事前 backup 有）。
- **モデル**: メイン=最新 Sonnet／alias(opus/fable/sonnet/haiku 最新)／軽作業=Haiku／`discover_models:false`+models で最新4種限定。
- **人格**: NEXUS → **Optimus**（🔱）。SOUL に Identity 追記、memory 改名。
- **AGENTS.md**: Hermes 向けに整理し `~/optimus/AGENTS.md`＋`terminal.cwd` 設定（Context 読込）。
- **MCP 4/5 稼働**: github / officecli / playwright / drawio（**aws-mcp は認証保留**）。
- **常駐化（ログオン不要）**: Task Scheduler `KeepWSLHermes`（AtStartup・run whether logged on or not）＋電源スリープ/休止無効で 24/7。`optimus` コマンドで WSL 一発ログイン。
- **Discord 自動起動**: Windows Startup フォルダにショートカット（ログイン後に自動起動）。
- **検証**: Optimus が自己申告（人格/記憶/AGENTS/モデル/MCP）＋ github 実取得で end-to-end 確認済み。

## 2. 残タスク

- **cron 再作成**: OpenClaw の定期ジョブを `hermes cron create` で移植。
  - 移植候補（汎用定期）: github-trending / hackernews / claude・aws automation-knowledge / monthly-windows-update
  - 環境依存で要判断: al2023-security(EC2 OS依存) / claude-cli-autoupdate(不要) / openclaw-autoupdate-notify(→hermes update)
  - 不要: taskboard-poller
  - 個人リマインダー(career): 近日分は当面 OpenClaw 維持
  - ⚠️ 二重発火注意（並行稼働）→ 移植後は OpenClaw 側を無効化。
- **aws-mcp 認証**: `~/.aws/credentials`（EC2 は instance role だったため WSL は明示認証が必要）。
- **軽微**: `.env` の旧 `MESSAGING_CWD`（存在しない OpenClaw パス）削除。
- **任意**: playwright-mcp の実疎通確認。

## 3. 引き継ぎ方針（重要）

**以降の作業は Hermes（Optimus）側で実施する。**
- 本ドキュメントまでの構築記録は OpenClaw 側からの作業として public リポ `docs/hermes/`（001〜006）に集約済み。
- 残タスク（cron 等）以降は Hermes 上の Optimus を主体に進め、必要に応じ Hermes 側でドキュメントを継続する。

## 4. 切り戻し・安全網

- 移行前スナップショット: `~/.hermes/backups/pre-migration-*.zip`（`hermes import` で復元）。
- WSL 全体バックアップ: `D:\wsl-backup\Ubuntu-full-*.tar`。
- 常駐化・ショートカットは可逆（`Unregister-ScheduledTask` / VBS・ショートカット削除・プロファイル該当行削除）。
