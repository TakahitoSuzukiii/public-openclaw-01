# AL2023 セキュリティアドバイザリ日次まとめ（2026-06-08）

作成日: 2026-06-08 / STATUS: INFO / TOPIC: ALAS / 対象: Critical+Important

> ALAS（Amazon Linux Security Advisory）= Amazon Linux 向けに AWS が公開するセキュリティ勧告。本記事は AWS 公式 ALAS フィードを情報源に、当日新規分のうち Critical（緊急）/ Important（重要）を抜粋して日本語要約したものです。CVE（Common Vulnerabilities and Exposures = 共通脆弱性識別子）番号を併記します。

## 概要（本日の新規アドバイザリ）

- 新規合計: **76 件**
- 重大度別: Critical **0** / Important **50** / Medium **26** / Low **0**
- 本記事の対象（Critical+Important）: **50 件**

> 注記: 本日の Critical/Important は **50 件**あり、本記事はフィードから詳細を取得できた**先頭 30 件**を掲載しています（残り 20 件も同日付の Important）。件数が多いため、特に影響範囲が広い PostgreSQL・各種コンテナ基盤・NVIDIA ドライバを中心に解説します。

---

## 注目アドバイザリ（影響が大きいもの）

### 1. PostgreSQL（postgresql15/16/17/18）— ALAS2023-2026-1766/1767/1768/1780

- **重大度:** Important
- **対象パッケージ:** postgresql17, postgresql16, postgresql15, postgresql18（全系統に修正提供）
- **主な CVE:** CVE-2026-6472, CVE-2026-6473〜6479, CVE-2026-6575, CVE-2026-6637, CVE-2026-6638
- **要約:** PostgreSQL（オープンソースの関係データベース）に複数の脆弱性。`CREATE TYPE` の認可不備（CVE-2026-6472）により、オブジェクト作成者が `search_path` 経由で型を解決する他者のクエリを乗っ取り、被害者の権限で任意の SQL 関数を実行させられます。さらに整数オーバーフロー（整数の桁あふれ）により、非特権ユーザがメモリ割り当てを過小化させて領域外書き込みを起こし、DB を動かす OS ユーザ権限で任意コード実行に至る可能性があります。修正版は 18.4 / 17.10 / 16.14 / 15.18 / 14.23。

### 2. コンテナ基盤（docker / containerd / nerdctl / ecs-init）— ALAS2023-2026-1783/1784/1788/1771

- **重大度:** Important
- **対象パッケージ:** docker, containerd, nerdctl, ecs-init（Amazon ECS 用初期化パッケージ）
- **主な CVE:** CVE-2026-39827〜39835, CVE-2026-42306, CVE-2026-42508, CVE-2026-42499, CVE-2026-46595/46597/46598, CVE-2024-45337, CVE-2026-33811, CVE-2026-33814 ほか
- **要約:** Go 言語の標準/準標準ライブラリ（特に `golang.org/x/crypto/ssh` と `net/http2`）に起因する一連の脆弱性が、コンテナ関連ツール群へ波及。SSH の公開鍵パーサがサイズ上限を課しておらず、巨大な RSA/DSA 鍵で署名検証に数分の CPU を消費させられる（CVE-2026-39829）、HTTP/2 の `SETTINGS_MAX_FRAME_SIZE=0` で CONTINUATION フレーム送信の無限ループに陥る（CVE-2026-33814）、拒否したチャネルの繰り返しでメモリが無制限に増大しサーバが落ちる（CVE-2026-39827）など、主に DoS（サービス妨害）系。コンテナ実行環境を広く使う構成では影響範囲が大きいパッケージ群です。

### 3. Apache Tomcat（tomcat9 / tomcat10）— ALAS2023-2026-1770/1776

- **重大度:** Important
- **対象パッケージ:** tomcat9, tomcat10
- **主な CVE:** CVE-2026-41284, CVE-2026-41293, CVE-2026-42498, CVE-2026-43512〜43515
- **要約:** Java サーブレットコンテナ Apache Tomcat に、リソースのスロットリング不備（CVE-2026-41284）や入力検証不備（CVE-2026-41293）など複数の脆弱性。影響範囲は 11.0.0-M1〜11.0.21 / 10.1.0-M1〜10.1.54 / 9.0.0.M1〜9.0.117 など。AWS が提供する修正版へのアップグレードが推奨されます。

### 4. NVIDIA Display Driver / CUDA 関連（17 アドバイザリ）— ALAS2023NVIDIA-2026-280〜296

- **重大度:** Important
- **対象パッケージ:** nvidia-driver, nvidia-open, kmod-nvidia-latest-dkms, kmod-nvidia-open-dkms, nvidia-fabricmanager, nvidia-imex, nvidia-kmod-common, nvidia-modprobe, nvidia-persistenced, nvidia-settings, nvidia-xconfig, nvlink5, nvlink5-580, libnvidia-nscq, libnvsdm, cuda-drivers, cuda-compat
- **主な CVE:** CVE-2025-33221, CVE-2026-24182, CVE-2026-24187, CVE-2026-24190, CVE-2026-24192〜24199（各アドバイザリ 11〜12 件）
- **要約:** NVIDIA Display Driver（Windows/Linux 向け）のカーネルドライバに、重要リソースへの不適切な権限割り当て（CVE-2025-33221）や、保持中のドライバロックの漏えい（CVE-2026-24182）、use-after-free（解放済みメモリの使用、CVE 群）など複数の脆弱性。悪用されると DoS、データ改ざん、権限昇格につながる恐れがあります。GPU を使う EC2 インスタンス（NVIDIA ドライバ導入済み）が対象。17 件は同一の CVE セットを共有する関連パッケージ群です。

---

## その他の Important（詳細掲載分）

- **ALAS2023-2026-1773 / nginx** — CVE-2026-9256。`ngx_http_rewrite_module` で、重複・入れ子になる PCRE（Perl 互換正規表現）キャプチャを使う `rewrite` ディレクティブと特定の置換文字列の組み合わせに脆弱性。
- **ALAS2023-2026-1777 / gnutls** — CVE-2026-33845。DTLS の再構築処理でリモートから誘発可能なアンダーフローによりヒープ領域のオーバーランが発生。
- **ALAS2023-2026-1782 / papers** — CVE-2026-46529。Evince / Atril / Xreader 系の `ev_spawn()` でシェル様入力のクオート漏れによるコマンドインジェクション。バンドルされた Rust 製 `rand` クレートの健全性問題（RUSTSEC-2026-0097）も併修。

---

## Medium の新規（参考・26 件）

本日は Medium が 26 件新規追加されました（Low は 0 件）。主なパッケージ例: python3.12/3.13/3.14/3.9, perl 系（perl, perl-libwww-perl, perl-HTTP-Tiny ほか）, gnutls, libssh, libssh2, mariadb1011, memcached, jq, sendmail, composer, bouncycastle, capstone, libsoup3, gstreamer1-plugins-good など。詳細は各 ALAS ページを参照してください。

---

## 対処方法（参考 — 実行は管理者）

以下は AL2023 公式の更新手順です。**いずれも sudo（管理者権限）が必要で、本監視タスクでは実行しません。** 適用是非・タイミングは管理者がご判断ください。

```bash
# リリース更新の有無を確認
sudo dnf check-release-update

# 更新可能なパッケージを確認
sudo dnf check-update

# セキュリティ更新のみ適用
sudo dnf upgrade --security
```

出典: [Managing repositories and OS updates（AWS 公式・日本語）](https://docs.aws.amazon.com/ja_jp/linux/al2023/ug/managing-repos-os-updates.html)

> NVIDIA ドライバや特定パッケージは利用構成によって更新方法・再起動要否が異なる場合があります。本番環境では検証環境での確認を推奨します。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
