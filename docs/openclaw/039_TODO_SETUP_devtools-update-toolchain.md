# 039 開発ツールチェーン最新化（Python / uv・uvx / Go）

- **ステータス:** TODO（uv・Python 完了 / Go は別途リマインドで実施予定）
- **対象サーバ:** Amazon Linux 2023（AL2023）
- **由来:** NEXUS タスクボード #91「サーバ保守: 開発ツール最新化（Python / uv・uvx / Go）」
- **実施日:** 2026-06-29（JST）
- **方針（重要）:** OS 標準 Python 3.9 は**置き換えない**（AL2023 のシステムツールが依存。`rpm --whatrequires /usr/bin/python3` に ec2-utils・nfs-utils 等が連なる）。新版は**併設**。既定 `python3` の切替は行わない。sudo を伴う変更は本人作業。

---

## 1. 着手時の現状と公式最新版（2026-06-29 確認）

| ツール | 現状 | 公式最新 | 導入元 |
|---|---|---|---|
| Python | 3.9.25（OS既定 `/usr/bin/python3`） | 3.13.14 / 3.14.6 | dnf @amazonlinux ほか |
| uv / uvx | 0.11.17（standalone `~/.local/bin`） | 0.11.25 | astral standalone |
| Go | 1.25.10（`/usr/bin/go`・dnf golang・Amazon Linux vendor） | go1.26.4 | dnf golang |

- 公式情報源: python.org（endoflife.date API で確認）/ astral-sh/uv（GitHub Releases）/ go.dev（`/dl/?mode=json`）。
- dnf には併設用に `python3.11` / `python3.12` / `python3.13(3.13.13)` パッケージあり。uv 管理なら `cpython-3.13.14` を取得可能。

---

## 2. 実施内容（完了分）

### 2-1. uv / uvx 更新（ユーザ領域・sudo不要）
```bash
~/.local/bin/uv self update      # 0.11.17 -> 0.11.25
uv --version && uvx --version    # => 0.11.25
```

### 2-2. Python 3.13 併設（uv 管理・sudo不要・OS無改変）
```bash
uv python install 3.13           # cpython-3.13.14 を取得
# 導入先: ~/.local/share/uv/python/cpython-3.13-linux-x86_64-gnu/
# シンボリックリンク: ~/.local/bin/python3.13 -> 上記 bin/python3.13
python3.13 --version             # => Python 3.13.14
uv run --python 3.13 -- python -V
```
- プロジェクトでは `uv run --python 3.13 …` もしくは `~/.local/bin/python3.13` で利用。
- 既定 `python3` は **3.9.25 のまま**（切替なし）。

### 2-3. 回帰確認（完了分）
```bash
python3 --version                # => 3.9.25（不変）
readlink -f "$(command -v python3)"  # => /usr/bin/python3.9
systemctl --user is-active openclaw-taskboard.service   # => active
```
- 既定 Python・常駐サービス（タスクボード）に影響なし。cron / MCP もユーザ領域変更のため影響なし。

---

## 3. 残作業（TODO・別途リマインドで実施）

### 3-1. Go を go1.26.4 へ（公式手動・sudo必要＝本人作業）
方針: go.dev 公式 tar を `/usr/local/go` へ展開し PATH 整理。既存 dnf golang（`/usr/bin/go`）と二重になるため、PATH 優先か dnf 版削除で整理する。

手順案（着手前に最新版を再確認すること）:
```bash
# 1) 取得 + チェックサム照合（go.dev/dl の sha256 と一致確認）
curl -LO https://go.dev/dl/go1.26.4.linux-amd64.tar.gz
sha256sum go1.26.4.linux-amd64.tar.gz

# 2) 展開（sudo）
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.26.4.linux-amd64.tar.gz

# 3) PATH 整理：/usr/local/go/bin を /usr/bin より優先
#    （既存 dnf golang と重複。dnf 版削除 or PATH 優先で一本化）
#    例: ~/.bashrc 等で export PATH=/usr/local/go/bin:$PATH

# 4) 検証 + 回帰
go version                       # => go1.26.4
```
- リマインド cron: `remind-task91-go-1.26.4`（2026-06-29 21:00 JST 単発）。
- 実施後、本ドキュメントの STATUS を DONE に更新し、Go セクションを実績へ書き換える。

---

## 4. メモ
- uv 管理 Python は OS と独立した standalone ビルド。OS の dnf 更新・セキュリティ更新の影響を受けないため、点リリース追従は `uv python upgrade` 等で別途運用する。
- マスキング規約に従い、ホームディレクトリは `~/`、OS ユーザ名は表記しない。本作業に機密値（トークン等）は登場しない。
