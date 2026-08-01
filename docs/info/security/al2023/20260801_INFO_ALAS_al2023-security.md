# AL2023 セキュリティアドバイザリ 週次まとめ（2026-08-01）

作成日: 2026-08-01 / STATUS: INFO / TOPIC: ALAS / 対象: Critical+Important

Amazon Linux 2023（AL2023）の公式セキュリティアドバイザリ（ALAS = Amazon Linux Security Advisory）を週次で監視した結果です。前回（2026-07-25）以降に新規公開されたアドバイザリを対象にしています。

## 概要

今週の新規アドバイザリは **3 件**、いずれも重大度 **Important（重要）** でした。

| 重大度 | 件数 |
|---|---|
| Critical（緊急） | 0 |
| Important（重要） | 3 |
| Medium（中） | 0 |
| Low（低） | 0 |

3 件はすべて Linux カーネル（OS の中核ソフトウェア）に関するもので、修正対象のカーネルバージョン（`kernel6.18` / `kernel6.12` / `kernel`）ごとに分割されていますが、**根本原因は同一の脆弱性 `CVE-2026-64600`** です。CVE（Common Vulnerabilities and Exposures）は、脆弱性ごとに世界共通で振られる識別番号を指します。

## 新規アドバイザリ（Important）

### 1. [ALAS2023-2026-2003](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-2003.html)

- **重大度:** Important（重要）
- **対象パッケージ:** `kernel6.18`（カーネル 6.18 系）
- **関連 CVE:** CVE-2026-64600
- **概要:** Linux カーネルの XFS ファイルシステム（大容量向けのファイルシステム）に存在する不具合の修正です。`ILOCK`（inode ロック = ファイル管理情報の排他ロック）を一度解放して取り直した後に、データフォーク（ファイル本体データの配置情報）のマッピングを再取得していなかったため、古い（stale な）情報を参照してしまう問題が指摘されました。ファイルの整合性やシステムの安定性に影響し得るため、修正が取り込まれています。

### 2. [ALAS2023-2026-2004](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-2004.html)

- **重大度:** Important（重要）
- **対象パッケージ:** `kernel6.12`（カーネル 6.12 系）
- **関連 CVE:** CVE-2026-64600
- **概要:** 上記 2003 と同一の脆弱性（XFS のデータフォーク・マッピング再取得漏れ）を、カーネル 6.12 系向けに修正したものです。内容は 2003 と同じで、適用対象のカーネルバージョンが異なります。

### 3. [ALAS2023-2026-2005](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-2005.html)

- **重大度:** Important（重要）
- **対象パッケージ:** `kernel`（標準カーネル）
- **関連 CVE:** CVE-2026-64600
- **概要:** こちらも同一の脆弱性（CVE-2026-64600）に対する、標準カーネルパッケージ向けの修正です。多くの AL2023 環境が使う既定のカーネルが対象になるため、実運用上はこの `kernel` パッケージの更新が特に関係します。

## 対処方法（参考・実行は管理者）

以下は AL2023 公式手順です。**いずれも sudo（管理者権限）が必要なため、本監視タスクでは実行しません。** 実際の適用は鈴木さん（管理者）の判断でお願いします。

```bash
# 利用可能なリリース更新の確認
sudo dnf check-release-update

# 更新可能なパッケージの確認
sudo dnf check-update

# セキュリティ更新のみ適用
sudo dnf upgrade --security
```

カーネル更新を反映するには、適用後に再起動が必要になる場合があります。

出典: <https://docs.aws.amazon.com/ja_jp/linux/al2023/ug/managing-repos-os-updates.html>

## Medium / Low の新規

今週、Medium・Low の新規アドバイザリはありませんでした。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
