# Playwright ブラウザ依存ライブラリ／Chrome 導入手順（ヘッドレス）

> **ステータス: 実施済み（DONE）** — 2026-06-07 に導入完了。本書は当初の調査・計画に、実際に行った導入手順（実施記録）を追記したもの。
> 設定そのものの解説は [playwright-mcp-setup.md](./006_DONE_SETUP_playwright-mcp-setup.md) を参照。

OpenClaw の **playwright-mcp** でブラウザ操作（ヘッドレス）を動かすために必要な OS 依存ライブラリと Chrome 本体の導入手順をまとめる。

---

## 0. 実施サマリ（2026-06-07）

| 項目 | 結果 |
|---|---|
| OS 依存ライブラリ（16個）の dnf 導入 | ✅ 完了 |
| `npx playwright install chrome` での Chrome 導入 | ❌ amzn 非対応で失敗（後述） |
| Google Chrome 安定版を **Google 公式 dnf リポジトリ**から導入 | ✅ 完了（`/opt/google/chrome/chrome`） |
| `--browser chrome` で playwright-mcp 起動・navigate 検証 | ✅ 成功 |
| バンドル版 Chromium のクリーンアップ | ✅ 実施（約636MB 解放） |

**最終構成**: `--browser chrome`（システムの Google Chrome 安定版を使用）。

---

## 1. 背景・目的

- サーバ（Amazon Linux 2023）上で **GUI を開く予定はない**。
- やりたいこと: **Playwright のヘッドレスモードで、ブラウザ操作を擬似的にテスト**する。
- 方針: **最小構成・最小モジュール**が理想。

## 2. なぜ依存ライブラリが必要か（GUI とは無関係）

ブラウザを起動できる状態のバイナリが無いと、`ldd` で確認したとき共有ライブラリが多数 `not found` になる。

重要な点:
- これらのライブラリは **「画面を描画するため」ではなく、Chrome バイナリが起動時に動的リンクしている共有ライブラリ**。1つでも欠けると **プロセスが起動すらできない**。
- **ヘッドレス** = 実行時に画面 / X サーバが不要、という意味。**バイナリがリンクする `.so` ファイル自体はディスク上に必要**。
- つまり「GUI を使わない」ことと「この lib 群が要る」ことは両立する。

## 3. 不足ライブラリ → AL2023 パッケージ対応表

`dnf provides` で裏取りした対応（すべて Amazon Linux 2023 標準リポジトリに存在）:

| 共有ライブラリ | パッケージ |
|---|---|
| libX11.so.6 | libX11 |
| libxcb.so.1 | libxcb |
| libXcomposite.so.1 | libXcomposite |
| libXdamage.so.1 | libXdamage |
| libXext.so.6 | libXext |
| libXfixes.so.3 | libXfixes |
| libXrandr.so.2 | libXrandr |
| libasound.so.2 | alsa-lib |
| libatk-1.0.so.0 | atk |
| libatk-bridge-2.0.so.0 | at-spi2-atk |
| libatspi.so.0 | at-spi2-core |
| libgbm.so.1 | mesa-libgbm |
| libxkbcommon.so.0 | libxkbcommon |
| libcairo.so.2 | cairo |
| libcups.so.2 | cups-libs |
| libpango-1.0.so.0 | pango |

---

## 4. 実施手順（実際に行った導入）

### 手順 1: OS 依存ライブラリの導入（要 sudo・管理者が実行）

```bash
sudo dnf install -y \
  libX11 libxcb libXcomposite libXdamage libXext libXfixes libXrandr \
  alsa-lib atk at-spi2-atk at-spi2-core mesa-libgbm libxkbcommon \
  cairo cups-libs pango
```

### 手順 2: Chrome 本体の導入

#### ⚠️ うまくいかない方法: `npx playwright install chrome`
Playwright のインストーラは **Ubuntu / Debian 専用**で、Amazon Linux（`amzn`）では次のように明示的に拒否される:

```
ERROR: cannot install on amzn distribution - only Ubuntu and Debian are supported
```

→ AL2023 ではこの方法は使えない。

#### ✅ 採用した方法: Google 公式 dnf リポジトリから導入（要 sudo）

```bash
sudo tee /etc/yum.repos.d/google-chrome.repo > /dev/null <<'EOF'
[google-chrome]
name=google-chrome
baseurl=https://dl.google.com/linux/chrome/rpm/stable/x86_64
enabled=1
gpgcheck=1
gpgkey=https://dl.google.com/linux/linux_signing_key.pub
EOF

sudo dnf install -y google-chrome-stable
```

- Google Chrome 安定版が **`/opt/google/chrome/chrome`** に配置される。
- playwright-mcp の `--browser chrome` がこのパスを参照する。

### 手順 3: playwright-mcp の設定確認

`~/.openclaw/openclaw.json` の playwright-mcp 設定:

```json
"playwright-mcp": {
  "command": "/usr/bin/npx",
  "args": ["-y", "@playwright/mcp@latest", "--headless", "--browser", "chrome"]
}
```

> `--browser chrome` = システムにインストールされた Google Chrome 安定版チャネルを使う指定。バンドル版 Chromium（Chrome for Testing）を使う場合は `chromium` にする。

---

## 5. 導入後の検証手順

```bash
# 1) Chrome 本体とバージョン確認
ls -l /opt/google/chrome/chrome
/opt/google/chrome/chrome --version    # 例: Google Chrome 149.x

# 2) ヘッドレス実起動テスト（DOM が出れば成功）
/opt/google/chrome/chrome --headless=new --no-sandbox --disable-gpu \
  --dump-dom https://example.com | grep -i '<title>'

# 3) playwright-mcp 経由で navigate / snapshot を実行し、ページ取得を確認
```

検証実績: Chrome 149 起動成功、playwright-mcp 経由で `https://example.com` の navigate に成功。

---

## 6. クリーンアップ（不要モジュール削除）

`--browser chrome` 採用により、playwright バンドルの Chromium は不要。削除して容量を解放した。

```bash
cd ~/.cache/ms-playwright
rm -rf chromium-1223 chromium_headless_shell-1223 mcp-chrome-for-testing-473d23d
```

- 結果: キャッシュ 643MB → 7.4MB（約 636MB 解放）。
- **残すもの**: `mcp-chrome-*`（chrome の実プロファイル・使用中）、`ffmpeg-*`（録画用）、`.links`（レジストリ）。

---

## 7. 参考: 当初検討した代替案

| 案 | 内容 | 採否 |
|---|---|---|
| 案A | 依存導入 + `--browser chromium`（バンドル版 Chrome for Testing 使用） | 不採用（chrome 本体を選択） |
| 案B | headless shell 狙いで依存3個削減 | 不採用（効果薄・要検証） |
| 案C/2b | 実 Google Chrome を導入し `--browser chrome` | **採用** |
| 案D | 保留（何もしない） | 当初方針→今回実施に移行 |

> どの案でも Playwright ヘッドレスのブラウザ操作（navigate / click / type / DOM 取得 / screenshot / PDF）は実現可能。今回は実 Chrome の挙動に寄せるため案C/2b を採用した。

---

## 8. 関連リンク

- Playwright MCP: <https://github.com/microsoft/playwright-mcp>
- Playwright（システム要件）: <https://playwright.dev/docs/browsers>
- 関連手順書: [playwright-mcp-setup.md](./006_DONE_SETUP_playwright-mcp-setup.md)

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
