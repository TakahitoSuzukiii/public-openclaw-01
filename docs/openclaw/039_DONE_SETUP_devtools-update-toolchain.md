# 039 開発ツールチェーン最新化（Python / uv・uvx / Go）

- **ステータス:** DONE（uv・Python・Go すべて完了）
- **対象サーバ:** Amazon Linux 2023（AL2023）
- **由来:** NEXUS タスクボード #91「サーバ保守: 開発ツール最新化（Python / uv・uvx / Go）」
- **実施日:** 2026-06-29（JST）
- **方針（重要）:** OS 標準 Python 3.9 は**置き換えない**（AL2023 のシステムツールが依存。`rpm --whatrequires /usr/bin/python3` に ec2-utils・nfs-utils 等が連なる）。新版は**併設**。既定 `python3` の切替は行わない。sudo を伴う変更は本人作業。

---

## 1. 着手時の現状と公式最新版（2026-06-29 確認）

| ツール | 着手時 | 公式最新 | 導入元 |
|---|---|---|---|
| Python | 3.9.25（OS既定 `/usr/bin/python3`） | 3.13.14 / 3.14.6 | dnf @amazonlinux ほか |
| uv / uvx | 0.11.17（standalone `~/.local/bin`） | 0.11.25 | astral standalone |
| Go | 1.25.10（`/usr/bin/go`・dnf golang・Amazon Linux vendor） | go1.26.4 | dnf golang |

- 公式情報源: python.org（endoflife.date API で確認）/ astral-sh/uv（GitHub Releases）/ go.dev（`/dl/?mode=json`）。

---

## 2. 実施内容（すべて完了）

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

### 2-3. Go 1.26.4（公式手動・/usr/local/go・要 sudo＝本人実施）
方式: go.dev 公式 tar を `/usr/local/go` へ展開し、PATH 優先で一本化（**案A**: dnf golang は残置）。
```bash
# 1) 取得 + チェックサム照合（go.dev 掲載 sha256 と一致確認・OK）
cd /tmp
curl -LO https://go.dev/dl/go1.26.4.linux-amd64.tar.gz
sha256sum go1.26.4.linux-amd64.tar.gz
#  => 1153d3d50e0ac764b447adfe05c2bcf08e889d42a02e0fe0259bd47f6733ad7f（公式一致）

# 2) 展開（sudo）
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf /tmp/go1.26.4.linux-amd64.tar.gz

# 3) PATH 優先（案A: dnf 版は残置、~/.bashrc で /usr/local/go/bin を優先）
echo 'export PATH=/usr/local/go/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# 4) 検証（実績）
hash -r
which go        # => /usr/local/go/bin/go
go version      # => go version go1.26.4 linux/amd64
go env GOROOT   # => /usr/local/go

# 5) 後片付け
rm -f /tmp/go1.26.4.linux-amd64.tar.gz
```
- **注意（案Aの仕様）:** `~/.bashrc` を読まない実行経路（cron / systemd 等）では従来の `/usr/bin/go`(dnf 1.25.10) が使われる。全経路で 1.26.4 に統一したい場合は案B（`/etc/profile.d/go.sh` ＋ `sudo dnf remove golang`）へ寄せる。今回は影響最小の案Aを採用。

---

## 3. 回帰確認（実施済み）
```bash
python3 --version                # => 3.9.25（不変）
readlink -f "$(command -v python3)"  # => /usr/bin/python3.9
systemctl --user is-active openclaw-taskboard.service   # => active
```
- 既定 Python・常駐サービス（タスクボード）に影響なし。uv/Python はユーザ領域、Go は PATH 優先（案A）で既存 dnf 版を残置のため、システム破壊リスクなし。

---

## 4. メモ
- uv 管理 Python は OS と独立した standalone ビルド。点リリース追従は `uv python upgrade` 等で別途運用。
- Go は将来全経路統一が必要になれば案B（profile.d ＋ dnf 版削除）へ移行可。
- マスキング規約に従い、ホームディレクトリは `~/`、OS ユーザ名・ホスト名は表記しない。本作業に機密値（トークン等）は登場しない。
