# OpenClaw → Hermes 移行 実行記録（Win11 / WSL）

> ステータス: DONE（主要部）/ カテゴリ: SETUP / 実施日 2026-08-08
> ⚠️ マスキング規約準拠。秘密（トークン/鍵）は一切記載しない。ホスト名/ユーザー名/IP/Discord ID 等は placeholder。
> 関連: `001_DONE_SETUP_hermes-agent-discord-wsl.md`（Discord 連携）／公式移行ガイド https://hermes-agent.nousresearch.com/docs/guides/migrate-from-openclaw/

## 0. 概要

既存 **OpenClaw**（別ホスト）の設定・データを、Windows 11 の **WSL 上の Hermes Agent** へ移行した記録。エージェント名は移行を機に **NEXUS → Optimus** へ改名（人格は継承）。OpenClaw は当面**並行稼働**のまま。

- 移行元: OpenClaw（`~/.openclaw` ＋ workspace）
- 移行先: Hermes（`~/.hermes`、WSL/Ubuntu）
- 方式: 公式 `hermes claw migrate` ＋ 手動補完（人格・AGENTS・モデル制限・MCP）

## 1. メインデータを D ドライブへ（WSL ディストロ移設）

`/mnt/d` 直置きは drvfs で SQLite ロック/性能/権限のリスク → **ディストロの `ext4.vhdx` を D: へ移設**する方式を採用。

```powershell
# バックアップ（フル）
wsl --shutdown
wsl --export Ubuntu D:\wsl-backup\Ubuntu-full-<date>.tar
# 移設（stopped を確認してから）
wsl --manage Ubuntu --move D:\WSL\Ubuntu
```
- `--manage --move` は既定ユーザー等を保持（root 化なし）。
- WSL 再起動耐性: `/etc/wsl.conf` に `[boot] systemd=true`（既設）。gateway は user systemd サービス＋linger で自動復帰。
- **注意**: WSL は Windows 起動時に自動起動しない → 常時起動化はタスクスケジューラでログオン時に WSL を常駐起動（`sleep infinity`）。

## 2. データ転送（機密はネット非経由）

- **機密・個人データ**（config・secrets・memory・個人資料）は **tar で固めてローカル DL → WSL で展開**（第三者サービス非経由）。byte-exact は sha256 で照合。
- 非機密の一般ドキュメントのみ GitHub を使用（本ドキュメント群）。
- WSL 側で `~/migration-src/.openclaw` に展開し、`--source` で移行に使用。

## 3. 移行実行（hermes claw migrate）

```bash
hermes claw migrate --source ~/migration-src/.openclaw --preset full --overwrite
```
- 衝突が1件でもあると全体適用拒否 → `--overwrite` を使用（item 単位バックアップ自動＋`~/.hermes/backups/pre-migration-*.zip` 生成、`hermes import` で復元可）。
- **Secrets は移行しない**（`--migrate-secrets` 不使用）。Discord は "No Discord settings found" で新 bot を完全保護。
- **結果: 13 件 migrated**（SOUL / MEMORY / USER / .env(非秘密) / model-config / skill / MCP×5 / agent-config）。
- archive 行き（手動対応）: IDENTITY / TOOLS / HEARTBEAT / cron / hooks 等。

## 4. モデル構成の検証（上書き後も無傷）

`--overwrite` 後も、事前に設定した Hermes モデル構成は保持されていることを確認:
- メイン: 最新 Sonnet ／ alias: sonnet・haiku・opus・fable（最新に固定）
- 補助（軽作業）: title_generation / compression / web_extract = 最新 Haiku
- terminal.backend = local

## 5. 人格移行と改名（NEXUS → Optimus）

- 人格・トーンは OpenClaw の `SOUL.md` を継承。
- `SOUL.md` 先頭に **Optimus の Identity ブロック**（Name/Role/Vibe/Emoji 🔱）を追記。
- `memories/MEMORY.md`・`USER.md` の "NEXUS" を "Optimus" へ置換。
- Discord で「Optimus」と名乗ることを確認。

## 6. AGENTS.md（運用ルール）の移植

Hermes は**作業ディレクトリの `AGENTS.md`** をセッション開始時に "# Project Context" として読み込む（既定 20,000 字上限・インジェクション安全チェックあり）。

- OpenClaw 固有ルール（タスクボード層／`/opt` 機構等）を除去し、**核心の運用ルール**（セキュリティ・HITL・マスキング・ドキュメント規約・報告スタイル・公式doc参照）を保持した **Hermes 向け AGENTS.md** を作成。
- 専用作業ディレクトリ `~/optimus/` に配置し、`terminal.cwd` をそこに設定。

```bash
hermes config set terminal.cwd /home/<your-user>/optimus
# ~/optimus/AGENTS.md を配置
```

## 7. モデルピッカーの制限（最新4種のみ）

`providers.anthropic` に `discover_models: false` ＋ models 列挙を追加し、ピッカーを最新世代のみに限定。

```yaml
providers:
  anthropic:
    discover_models: false
    models:
      - claude-opus-5
      - claude-sonnet-5
      - claude-haiku-4-5
      - claude-fable-5
```

## 8. MCP サーバー再セットアップ（5件）

移行された MCP は EC2 のバイナリ/パス前提だったため、WSL 上で実行体を新規導入。**home レイアウトが同一**なので path はほぼそのまま流用可（playwright のみ修正）。

| MCP | 実行体 | 導入方法 | 状態 |
|---|---|---|---|
| drawio-mcp | HTTP | 不要 | ✅ |
| github-mcp | `~/go/bin/github-mcp-server` | `go install github.com/github/github-mcp-server/cmd/github-mcp-server@latest` | ✅ |
| officecli | `~/.local/bin/officecli` | `curl -fsSL https://d.officecli.ai/install.sh \| bash` | ✅ |
| playwright-mcp | `npx` + Chrome | `npx playwright install --with-deps chrome`（command を実 npx パスへ修正） | ✅ |
| aws-mcp | `~/.local/bin/uvx` | `curl -LsSf https://astral.sh/uv/install.sh \| sh` | ⏸ AWS 認証待ち |

- **github トークン配線**（秘密は config に書かない）:
  ```bash
  hermes config set mcp_servers.github-mcp.env.GITHUB_PERSONAL_ACCESS_TOKEN '${env:GITHUB_TOKEN}'
  ```
  （`.env` の `GITHUB_TOKEN` を参照して MCP へ渡す）
- **aws-mcp**: EC2 ではインスタンスロールで認証していたため WSL では明示認証が必要。`~/.aws/credentials` を用意して `restart` で有効化予定（現在保留）。

## 9. 残タスク（次回）

- **cron 再作成**: OpenClaw の定期ジョブ（週次トレンド／HackerNews／Claude・AWS 自動化ナレッジ／月次 Windows Update 監視 等）を `hermes cron create` で再現。並行稼働のため二重発火に注意（移植後は OpenClaw 側を無効化）。career リマインダーは当面 OpenClaw 維持。
- **aws-mcp**: AWS 認証情報を用意して有効化。
- 軽微: `.env` の旧 `MESSAGING_CWD`（存在しない OpenClaw パス）を削除。

## 参考（公式）
- 移行: https://hermes-agent.nousresearch.com/docs/guides/migrate-from-openclaw/
- context files: https://hermes-agent.nousresearch.com/docs/user-guide/features/context-files/
- configuration: https://hermes-agent.nousresearch.com/docs/user-guide/configuration/
- cron: https://hermes-agent.nousresearch.com/docs/user-guide/features/cron/
