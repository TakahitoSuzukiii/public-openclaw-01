# 047 MCPサーバ起動失敗の対応（github-mcp / aws-mcp）＋ orphan transcript の整理

> STATUS: DONE / CATEGORY: SETUP / 作成日: 2026-07-19
> `openclaw doctor` が報告していた残課題2件（① MCPサーバ起動失敗 ② 孤立トランスクリプト）を、診断→対応した記録。読み取り診断は自由、書き込み（config変更・ファイル削除・再起動）は HITL 承認の上で実施した。マスキング規約に従い、実ホスト名・IP・実ユーザ名・トークン等の実値は記載しない（`<your-user>` 等の placeholder を使用）。

---

## 1. 背景（doctor の残課題）

- **github-mcp**: `openclaw doctor` で `[bundle-mcp] failed to start server "github-mcp" (... stdio): McpError -32000: Connection closed` が報告されていた。
- **aws-mcp**: `Request timed out`（uvx 経由ロード）が報告されていた。
- **orphan transcript**: `~/.openclaw/agents/main/sessions` に sessions.json から参照されない孤立 `.jsonl` が存在（doctor 実行時点で 3 件）。

---

## 2. 診断（読み取りのみ）

### 2-1. github-mcp
- バイナリ `~/go/bin/github-mcp-server` は実在・実行権限あり（755, 約27MB）。
- 手動で `github-mcp-server stdio` を起動 → `server session connected` まで到達し、**起動自体は成功**することを確認（単発入力の EOF で正常終了）。
- `--version` は `version` / `commit` のプレースホルダ表示（ビルド時に ldflags 未注入。**表示のみの問題で動作に無関係**）。
- 認証: `GITHUB_PERSONAL_ACCESS_TOKEN` は環境に設定済み（実値は記載しない）。
- **結論**: ハード故障ではなく、doctor の起動プローブでの **間欠的なタイミング競合**（初期化前に接続が閉じる）と判断。`mcp.servers.*` に `connectionTimeoutMs` の設定が無く、既定が短い可能性（同ファイルの drawio-mcp には `connectionTimeoutMs` が設定済みで、この knob が有効なことを確認）。

### 2-2. aws-mcp
- `uv` / `uvx` は導入済み。設定は `uvx mcp-proxy-for-aws@latest ...`（`@latest` で毎回バージョン解決が走る）。
- doctor 再実行時には aws-mcp の警告は出ず＝**間欠**。`uvx` の初回コールドスタート（DL/解決）に伴うタイムアウトと判断。

### 2-3. orphan transcript
- sessions.json のライブセッション参照と `.jsonl` 実ファイルを突合し、未参照の孤立ファイル **3 件** を特定（`*.trajectory.jsonl` / `*.deleted.*` / `*.reset.*` は対象外）。

---

## 3. 対応（HITL 承認済み）

### 3-1. config 変更（`~/.openclaw/openclaw.json` の `mcp.servers`・追加のみ）
既存キーは変更せず、起動タイムアウトを **追加のみ** で付与した。

| サーバ | 追加キー | 値 | 意図 |
|---|---|---|---|
| github-mcp | `connectionTimeoutMs` | `30000` | 起動プローブの猶予を増やし間欠失敗を抑制 |
| aws-mcp | `connectionTimeoutMs` | `45000` | uvx コールドスタート（初回解決）の猶予を確保 |

- 変更前に `openclaw.json` を `~/.openclaw/workspace/.backups/task105-20260719/openclaw.json.bak` へ退避。
- 変更後、JSON 妥当性を検証（`python3 -c "json.load(...)"` で OK）。差分は追加2キーのみ（デグレ無し）。
- 反映のため **gateway を restart**（事前に Discord 通知／`stop`+`start` ではなく `restart`）。

### 3-2. orphan transcript のアーカイブ
- 3 件を `~/.openclaw/workspace/.backups/task105-20260719/orphan-transcripts.tar.gz` へ tar.gz 退避（gzip 整合性チェック済み）。
- アーカイブ確認後に元ファイルを削除。
- `openclaw doctor` を再実行し、**orphan transcript の警告が解消**（クリーン）を確認。

---

## 4. 検証

- [x] orphan transcript: doctor で警告消失（クリーン）。
- [x] config: JSON 妥当・追加のみ・バックアップ取得済み。
- [x] gateway restart 後、`openclaw doctor` で github-mcp / aws-mcp の起動状況を再確認（間欠のため、以後も doctor で経過観察）。
- 機密ゼロ・追加のみ・デグレ無しを確認。

---

## 5. 今後の経過観察・追加オプション

- `connectionTimeoutMs` 付与後も github-mcp の間欠失敗が続く場合の追加案:
  - github-mcp をバージョン情報付きで再ビルド（`--version` 正常化。動作影響は無いが識別性向上）。
  - aws-mcp の `mcp-proxy-for-aws@latest` を**固定バージョンにピン**して再解決を排除（安定化）。
- いずれも「追加のみ・最小変更」を維持し、変更時は本ドキュメントを更新する。

---

## 6. 実施コマンド（要点・値はマスク）

```bash
# 診断
openclaw doctor
ls -la ~/go/bin/github-mcp-server
printf '{"jsonrpc":"2.0",...}' | ~/go/bin/github-mcp-server stdio   # 起動確認
uv --version; uvx --version

# orphan アーカイブ（HITL 承認後）
tar -czf <backup>/orphan-transcripts.tar.gz <orphan1>.jsonl <orphan2>.jsonl <orphan3>.jsonl
gzip -t <backup>/orphan-transcripts.tar.gz
rm -f <orphan1>.jsonl <orphan2>.jsonl <orphan3>.jsonl
openclaw doctor   # クリーン確認

# config（HITL 承認後・追加のみ）→ gateway restart
cp ~/.openclaw/openclaw.json <backup>/openclaw.json.bak
# mcp.servers.github-mcp.connectionTimeoutMs=30000 / aws-mcp.connectionTimeoutMs=45000 を追加
openclaw gateway restart   # 事前に Discord 通知
```

> 注記: 本作業のバックアップは `~/.openclaw/workspace/.backups/task105-20260719/`。切り戻しは `openclaw.json.bak` の復元＋restart、トランスクリプトは tar.gz の展開で可能。
