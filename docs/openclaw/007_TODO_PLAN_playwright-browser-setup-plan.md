# Playwright ブラウザ依存ライブラリ 導入計画（ヘッドレス）

> **ステータス: 未実行（保留中）** — 本書は調査・計画の記録。実際の導入作業は行っていない。将来ブラウザ自動化が必要になった際の判断材料として残す。

OpenClaw の **playwright-mcp** でブラウザ操作（ヘッドレス）を動かすために必要な OS 依存ライブラリの調査結果と、複数の作業計画をまとめる。設定そのものの解説は [playwright-mcp-setup.md](./006_DONE_SETUP_playwright-mcp-setup.md) を参照。

---

## 1. 背景・目的

- サーバ（Amazon Linux 2023）上で **GUI を開く予定はない**。
- やりたいこと: **Playwright のヘッドレスモードで、ブラウザ操作を擬似的にテスト**する。
- 方針: **最小構成・最小モジュール**が理想。

---

## 2. なぜ依存ライブラリが必要か（GUI とは無関係）

現状、サーバにはブラウザを起動できる状態のバイナリが無い。`ldd` で確認すると共有ライブラリが多数 `not found` になっている。

重要な点:

- これらのライブラリは **「画面を描画するため」ではなく、Chrome バイナリが起動時に動的リンクしている共有ライブラリ**。1つでも欠けると **プロセスが起動すらできない**。
- **ヘッドレス** = 実行時に画面 / X サーバが不要、という意味。**バイナリがリンクする `.so` ファイル自体はディスク上に必要**。
- つまり「GUI を使わない」ことと「この lib 群が要る」ことは両立する。lib がある＝起動できる、画面は出さない（ヘッドレス）。

---

## 3. 実測結果（不足ライブラリ件数）

Playwright が取得済みの2種類のバイナリで `ldd ... | grep "not found"` を実測:

| バイナリ | 不足 `.so` 件数 | 用途 |
|---|---|---|
| `chrome-headless-shell`（ヘッドレス専用ビルド） | **13** | ヘッドレス自動化に特化した軽量ビルド |
| `chrome`（フル Chromium） | **16** | 通常の Chromium。ヘッド/ヘッドレス両対応 |

差分は **3つだけ**（`cairo` / `cups-libs` / `pango`）。最小化で削れる余地は限定的。

---

## 4. 不足ライブラリ → AL2023 パッケージ対応表

`dnf provides` で裏取りした対応（すべて Amazon Linux 2023 標準リポジトリに存在）:

| 共有ライブラリ | パッケージ | headless shell | full chromium |
|---|---|:--:|:--:|
| libX11.so.6 | libX11 | ✓ | ✓ |
| libxcb.so.1 | libxcb | ✓ | ✓ |
| libXcomposite.so.1 | libXcomposite | ✓ | ✓ |
| libXdamage.so.1 | libXdamage | ✓ | ✓ |
| libXext.so.6 | libXext | ✓ | ✓ |
| libXfixes.so.3 | libXfixes | ✓ | ✓ |
| libXrandr.so.2 | libXrandr | ✓ | ✓ |
| libasound.so.2 | alsa-lib | ✓ | ✓ |
| libatk-1.0.so.0 | atk | ✓ | ✓ |
| libatk-bridge-2.0.so.0 | at-spi2-atk | ✓ | ✓ |
| libatspi.so.0 | at-spi2-core | ✓ | ✓ |
| libgbm.so.1 | mesa-libgbm | ✓ | ✓ |
| libxkbcommon.so.0 | libxkbcommon | ✓ | ✓ |
| libcairo.so.2 | cairo | — | ✓ |
| libcups.so.2 | cups-libs | — | ✓ |
| libpango-1.0.so.0 | pango | — | ✓ |

---

## 5. 作業計画（4パターン）

### 案A：標準ヘッドレス（推奨・最小で確実）
- **実現すること**: Playwright ヘッドレスでブラウザ操作テスト全般（navigate / click / type / DOM 取得 / screenshot / PDF）
- **導入**: 16パッケージ（すべて AL2023 標準・小サイズ）
- **設定変更**: `--browser chrome` → `--browser chromium`（同梱 chromium 使用、追加ダウンロードなし＝cache 済）
- 追加容量ほぼゼロ。最も確実。

### 案B：超最小（headless shell 狙い）
- **実現すること**: 案Aと同じ。依存を13個に削減（cairo / cups-libs / pango を省く）
- **代償**: Playwright に headless shell を使わせる指定が必要。最新 Playwright の `headless:true` はフル chromium を使う仕様で、@playwright/mcp 経由で headless shell を確実に使えるかは **要検証**。削減はわずか3 lib。
- 労力対効果は低め。

### 案C：実 Chrome（Chrome for Testing）
- **実現すること**: 案A + Google Chrome 固有挙動（一部メディアコーデック等）
- **導入**: 16パッケージ + playwright-mcp が Chrome for Testing を追加ダウンロード（容量増）
- 擬似テスト目的にはオーバースペック。

### 案D：保留（何もしない）← 現在の方針
- ブラウザ機能は使わない。Discord / 他 MCP（aws / github / drawio）は現状通り正常。

---

## 6. 推奨案A：導入コマンド（参考）

> 実行には `sudo` が必要。実施は管理者が行う。

```bash
sudo dnf install -y \
  libX11 libxcb libXcomposite libXdamage libXext libXfixes libXrandr \
  alsa-lib atk at-spi2-atk at-spi2-core mesa-libgbm libxkbcommon \
  cairo cups-libs pango
```

---

## 7. 導入後の検証手順

```bash
# 1) 不足ライブラリが解消されたか（出力が空ならOK）
ldd ~/.cache/ms-playwright/chromium-*/chrome-linux64/chrome | grep "not found"

# 2) 実起動テスト（バージョンが表示されれば起動成功）
~/.cache/ms-playwright/chromium-*/chrome-linux64/chrome --headless --no-sandbox --dump-dom about:blank | head

# 3) config を chromium に変更後、playwright-mcp 経由で navigate/snapshot を試す
```

検証で問題なければ `~/.openclaw/openclaw.json` の `--browser` を `chromium` に変更（ホットリロードで反映）。

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
