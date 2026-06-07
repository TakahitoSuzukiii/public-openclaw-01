# AL2023 セキュリティアドバイザリ（Critical 棚卸し・2026-06-07 初回）

> 作成日: 2026-06-07 / STATUS: INFO / TOPIC: ALAS / 対象: Critical
> 情報源: Amazon Linux Security Center（ALAS）公式 RSS <https://alas.aws.amazon.com/AL2023/alas.rss>
> 本記事は日次監視タスクの**初回（ベースライン）**として、現在 AL2023(Amazon Linux 2023) で **Critical** に分類されている全 **12 件**を棚卸ししたものです。翌日以降は「新規に公開された Critical/Important」を差分通知します。

> 用語: **ALAS** … Amazon Linux のセキュリティ修正情報（アドバイザリ）の公式公開先。`ALAS2023-YYYY-NNNN` の ID で管理。
> 用語: **CVE** … 脆弱性に世界共通で振られる識別番号（Common Vulnerabilities and Exposures）。
> 用語: **RCE（Remote Code Execution）** … 遠隔から任意のコードを実行されてしまう、最も深刻な部類の脆弱性。
> 用語: **DoS（Denial of Service）** … サービス妨害。処理を停止・過負荷にさせられる攻撃。

## 概要

- 重大度別件数（現時点のフィード全体）: **Critical=12** / Important=1125 / Medium=843 / Low=191。
- 本記事は上記 **Critical 12 件**を対象（新しい公開日順）。各項目に対象パッケージ・修正バージョン・CVE・概要・リンクを記載。
- **対処**: 後述の「対処方法」のコマンドで、該当パッケージが導入されていれば最新へ更新してください（実行は管理者）。導入していないパッケージのアドバイザリは影響しません。

---

## Critical アドバイザリ一覧（公開日が新しい順）

### 1. lasso — SAML 処理で任意コード実行の恐れ
- **ID**: [ALAS2023-2025-1285](https://alas.aws.amazon.com/AL2023/ALAS2023-2025-1285.html) ／ 公開: 2025-11-10
- **対象パッケージ / 修正版**: `lasso` → **2.9.0-1.amzn2023**
- **CVE**: CVE-2025-46404, CVE-2025-46705, CVE-2025-47151
- **概要**: SAML（シングルサインオンの規格）処理ライブラリ Lasso に複数の脆弱性。細工した SAML レスポンスにより DoS（CVE-2025-46404 / -46705）、さらに型混同による**任意コード実行**（CVE-2025-47151）に至る恐れ。SAML 認証を扱うシステムは影響大。

### 2. .NET 8.0 — 権限昇格 / HTTP リクエストスマグリング
- **ID**: [ALAS2023-2025-1230](https://alas.aws.amazon.com/AL2023/ALAS2023-2025-1230.html) ／ 公開: 2025-10-23
- **対象パッケージ / 修正版**: `dotnet8.0` → SDK **8.0.121** / ランタイム **8.0.21**（amzn2023）
- **CVE**: CVE-2025-55247, CVE-2025-55248, CVE-2025-55315
- **概要**: .NET のリンク解決の不備によるローカル権限昇格（-55247）、暗号強度不足による情報漏えい（-55248）、ASP.NET Core の HTTP リクエスト/レスポンススマグリングによるセキュリティ機能バイパス（-55315）。.NET アプリ稼働環境は要更新。

### 3. .NET 9.0 — 権限昇格 / HTTP リクエストスマグリング
- **ID**: [ALAS2023-2025-1231](https://alas.aws.amazon.com/AL2023/ALAS2023-2025-1231.html) ／ 公開: 2025-10-23
- **対象パッケージ / 修正版**: `dotnet9.0` → SDK **9.0.111** / ランタイム **9.0.10**（amzn2023）
- **CVE**: CVE-2025-55247, CVE-2025-55248, CVE-2025-55315
- **概要**: 上記 .NET 8.0 と同一の 3 件の脆弱性が .NET 9.0 系にも該当。内容・対処は #2 と同じ。

### 4. NVIDIA Container Toolkit — コンテナ初期化フックで権限昇格・RCE
- **ID**: [ALAS2023NVIDIA-2025-125](https://alas.aws.amazon.com/AL2023/ALAS2023NVIDIA-2025-125.html) ／ 公開: 2025-07-17
- **対象パッケージ / 修正版**: `nvidia-container-toolkit` → **1.17.8-1**
- **CVE**: CVE-2025-23266, CVE-2025-23267
- **概要**: コンテナ初期化に使うフックの脆弱性で、攻撃者が**昇格した権限で任意コード実行**しうる（-23266）。また update-ldcache フックでの link following によりデータ改ざん・DoS（-23267）。GPU コンテナ利用環境は要更新。

### 5. libnvidia-container — 同上（共通ライブラリ側）
- **ID**: [ALAS2023NVIDIA-2025-126](https://alas.aws.amazon.com/AL2023/ALAS2023NVIDIA-2025-126.html) ／ 公開: 2025-07-17
- **対象パッケージ / 修正版**: `libnvidia-container` → **1.17.8-1**
- **CVE**: CVE-2025-23266, CVE-2025-23267
- **概要**: #4 と同じ 2 件の脆弱性に対する、共通ライブラリ `libnvidia-container` 側の修正。Toolkit と併せて更新する。

### 6. Squid — HTTP Digest 認証の DoS
- **ID**: [ALAS2023-2023-402](https://alas.aws.amazon.com/AL2023/ALAS2023-2023-402.html) ／ 公開: 2023-11-03
- **対象パッケージ / 修正版**: `squid` → **5.8-1.amzn2023.0.1**
- **CVE**: CVE-2023-46847
- **概要**: プロキシサーバ Squid のバッファオーバーフローにより、HTTP Digest 認証に対する DoS が可能。Squid を運用している場合は更新を。

### 7. APR — 配列の境界外読み取り / 整数オーバーフロー
- **ID**: [ALAS2023-2023-016](https://alas.aws.amazon.com/AL2023/ALAS2023-2023-016.html) ／ 公開: 2023-03-22
- **対象パッケージ / 修正版**: `apr` → **1.7.2-2.amzn2023.0.2**
- **CVE**: CVE-2017-12613, CVE-2021-35940, CVE-2022-24963
- **概要**: Apache Portable Runtime（多くの Apache 系ソフトの基盤ライブラリ）に、境界外読み取りと base64 関数の整数オーバーフローによる境界外書き込み。依存ソフトが多いため影響範囲に注意。

### 8. NSS — 署名処理のヒープオーバーフロー
- **ID**: [ALAS2023-2023-031](https://alas.aws.amazon.com/AL2023/ALAS2023-2023-031.html) ／ 公開: 2023-03-22
- **対象パッケージ / 修正版**: `nss` → **3.83.0-1.amzn2023.0.2**（nspr 4.35.0）
- **CVE**: CVE-2021-43527
- **概要**: NSS（TLS/証明書処理ライブラリ）で、DER エンコードされた DSA/RSA-PSS 署名処理時にヒープオーバーフロー。CMS・S/MIME・証明書検証などで NSS を使うアプリが影響を受けうる。

### 9. expat — XML パーサの整数オーバーフロー多数
- **ID**: [ALAS2023-2023-058](https://alas.aws.amazon.com/AL2023/ALAS2023-2023-058.html) ／ 公開: 2023-03-22
- **対象パッケージ / 修正版**: `expat` → **2.5.0-1.amzn2023.0.2**
- **CVE**: CVE-2021-45960, CVE-2021-46143, CVE-2022-22822〜22827, CVE-2022-23852, CVE-2022-23990, CVE-2022-25235, CVE-2022-25236, CVE-2022-25313〜25315, CVE-2022-40674, CVE-2022-43680（計17件）
- **概要**: 広く使われる XML パーサ libexpat の多数の整数オーバーフロー等。細工した XML により異常終了や RCE の恐れ。expat に依存するソフトが非常に多いため要更新。

### 10. xmlrpc-c — 不正 UTF-8 経由の RCE（expat 由来）
- **ID**: [ALAS2023-2023-068](https://alas.aws.amazon.com/AL2023/ALAS2023-2023-068.html) ／ 公開: 2023-03-22
- **対象パッケージ / 修正版**: `xmlrpc-c` → **1.51.08-2.amzn2023.0.1**
- **CVE**: CVE-2022-25235
- **概要**: 不正な 2〜3 バイト UTF-8 シーケンスの処理（expat 由来）により任意コード実行に至る恐れ。XML-RPC を使う場合は更新を。

### 11. maven-shared-utils — コマンドインジェクション
- **ID**: [ALAS2023-2023-077](https://alas.aws.amazon.com/AL2023/ALAS2023-2023-077.html) ／ 公開: 2023-03-22
- **対象パッケージ / 修正版**: `maven-shared-utils` → **3.3.4-4.amzn2023.0.3**
- **CVE**: CVE-2022-29599
- **概要**: Maven の共通ユーティリティで、二重引用符のエスケープ不備によりシェルインジェクション（コマンドインジェクション）が可能。Maven ビルド環境は更新を。

### 12. ClamAV — ファイルパーサの RCE / 情報漏えい
- **ID**: [ALAS2023-2023-112](https://alas.aws.amazon.com/AL2023/ALAS2023-2023-112.html) ／ 公開: 2023-03-22
- **対象パッケージ / 修正版**: `clamav` → **0.103.8-1.amzn2023.0.1**
- **CVE**: CVE-2023-20032, CVE-2023-20052
- **概要**: アンチウイルス ClamAV の HFS+ ファイルパーサで RCE（-20032）、DMG パーサで情報漏えい（-20052）。ClamAV を使う環境は更新を。

---

## 対処方法（参考・実行は管理者）

該当パッケージが導入されている場合のみ影響します。AL2023 公式手順で確認・適用してください（`sudo` 権限が必要。本監視タスクは通知のみで適用は行いません）。

```bash
# 新しい AL2023 リリースが出ているか確認
sudo dnf check-release-update

# 更新可能なパッケージを確認
sudo dnf check-update

# セキュリティ更新のみを適用
sudo dnf upgrade --security

# 特定パッケージのみ更新する例
sudo dnf upgrade --security <package-name>
```

> 出典: [Amazon Linux 2023 ユーザーガイド「リポジトリと OS の更新の管理」](https://docs.aws.amazon.com/ja_jp/linux/al2023/ug/managing-repos-os-updates.html)
> 個別アドバイザリの「New Packages」に記載の正確な修正バージョン以上へ更新されていれば対処済みです。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
