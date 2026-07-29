# 050 OfficeCLI 導入 ＋ NEXUS(OpenClaw) への MCP 登録

- STATUS: DONE / CATEGORY: SETUP
- 対応日: 2026-07-29
- 対象: OpenClaw サーバ（AL2023 / x86_64）への OfficeCLI 導入と、`mcp.servers` への登録

## 概要

**OfficeCLI**（`iOfficeAI/OfficeCLI`）は、AIエージェント向けの Office 操作 CLI（Word/Excel/PowerPoint を Office 非依存・単一バイナリで作成/読取/編集）。本手順では ①バイナリを固定バージョンで導入し、②OpenClaw の MCP クライアントレジストリに登録して NEXUS から使えるようにする。

- 調査メモ: `docs/info/research/20260729_INFO_OFFICECLI_ai-agent-office-cli.md`

## 前提（確認済み）

- OS: Amazon Linux 2023 / アーキ: x86_64 / libc: glibc 2.34（→ 配布バイナリは `officecli-linux-x64`。alpine/musl 版ではない）
- 設置先: `~/.local/bin`（PATH 上・**sudo 不要**のユーザ領域）

## 手順①: バイナリ導入（監査可能・最小構成）

> `curl | bash` 一括インストールや npm postinstall は、エージェント設定を自動改変する副作用（skill 注入・MCP 自動登録）があるため採らず、**固定リリース＋SHA256 照合**で手動導入する。

```bash
VER=v1.0.143
BASE=https://github.com/iOfficeAI/OfficeCLI/releases/download/$VER
cd /tmp
curl -fsSL -o officecli-linux-x64 "$BASE/officecli-linux-x64"
curl -fsSL -o SHA256SUMS       "$BASE/SHA256SUMS"
# SHA256 照合（一致しなければ中止）
grep ' officecli-linux-x64$' SHA256SUMS
sha256sum officecli-linux-x64
# 設置
install -m 0755 officecli-linux-x64 ~/.local/bin/officecli
officecli --version                 # => 1.0.143
# バックグラウンド自動更新を無効化（固定運用）
officecli config autoUpdate false   # config: ~/.officecli/config.json
```

- 動作テスト（後始末込み）:
  ```bash
  officecli create /tmp/oc_test.pptx
  officecli add /tmp/oc_test.pptx / --type slide --prop title="Hello OfficeCLI"
  officecli view /tmp/oc_test.pptx outline
  rm -f /tmp/oc_test.pptx
  ```

## 手順②: NEXUS(OpenClaw) への MCP 登録

OfficeCLI は `officecli mcp`（引数なし）で **stdio MCP サーバ**として起動する。OpenClaw の MCP レジストリへは公式 CLI で登録する（openclaw.json 手編集より安全）。

```bash
# 事前に openclaw.json をバックアップ
cp -p ~/.openclaw/openclaw.json ~/.openclaw/workspace/.backups/openclaw.json.pre-officecli-mcp.<ts>.bak

# 登録（mcp.servers.officecli を追加）
openclaw mcp add officecli --command ~/.local/bin/officecli --arg mcp --timeout 30

# 確認
openclaw mcp show officecli
openclaw mcp probe officecli --json   # => tools: ["officecli__officecli"]（統合1ツール）
```

登録後の `mcp.servers.officecli`:
```json
{ "command": "~/.local/bin/officecli", "args": ["mcp"], "timeout": 30 }
```
（実体は絶対パスで保存される）

## 反映（ホットリロード／再起動）

- OpenClaw の設定ホットリロードでは **`mcp` は「Tools & media」カテゴリ＝再起動不要でホット適用**（`docs/gateway/configuration.md` の反映表）。→ 以降の**新規セッション**（cron/isolated 含む）は自動で officecli ツールを取得する。
- ただし**既に稼働中のライブセッション**へツールを射影するには、**gateway 再起動**（または新規セッション開始）が必要。本対応では NEXUS 本体からも即利用できるよう、**事前通知のうえ `openclaw gateway restart`** を実施した（`restart`、stop+start は使わない）。

## 検証

- `officecli --version` = 1.0.143、動作テスト（create/add/view outline）OK。
- `openclaw mcp probe officecli` = 1 tool（`officecli__officecli`）。
- gateway 再起動後、NEXUS セッションに `officecli__officecli` ツールが射影されることを確認。

## セキュリティ / 留意点

- 外部バイナリ実行は**固定版＋SHA256 照合**で担保。ユーザ領域（`~/.local/bin`）に隔離、sudo 不要。
- 自動更新は無効化（固定運用）。更新する場合は同じ照合手順で手動更新。
- MCP 追加によりツール（＝攻撃面）が増える。外部由来の文書を扱う際は **untrusted 扱い・HITL** を継続。
- OpenClaw は stdio MCP 起動時に危険な環境変数（`LD_*`/`NODE_OPTIONS` 等）をブロックする。

## 切り戻し

```bash
# MCP 登録解除
openclaw mcp unset officecli        # または openclaw.json バックアップから復元
# バイナリ削除
rm -f ~/.local/bin/officecli ~/.officecli/config.json
```
- config バックアップ: `~/.openclaw/workspace/.backups/openclaw.json.pre-officecli-mcp.<ts>.bak`

## 参考

- リポジトリ: <https://github.com/iOfficeAI/OfficeCLI> / サイト: <https://officecli.ai>
- 反映表: `docs/gateway/configuration.md`（What hot-applies vs restart）

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
