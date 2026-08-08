# AL2023 セキュリティアドバイザリ週次まとめ（2026-08-08）

作成日: 2026-08-08 / STATUS: INFO / TOPIC: ALAS / 対象: Critical+Important

> ALAS（Amazon Linux Security Advisory、Amazon Linux 向けの公式セキュリティ勧告）の週次監視。前回スナップショット（2026-08-01）との差分から、新規に公開された **Critical / Important** を抜粋してまとめます。情報源は AWS 公式 ALAS フィードのみ。実際のパッチ適用は sudo 権限が必要なため、本タスクでは実行していません。

## 今週の概要

- 新規アドバイザリ合計: **89 件**
  - Critical: **4 件**
  - Important: **80 件**
  - Medium: 5 件
  - Low: 0 件
- 報告対象（Critical + Important）: **84 件**

Important の 80 件は、すべて **kernel-livepatch**（カーネルライブパッチ＝再起動なしで当てるカーネル修正）で、対象 CVE は共通の 2 件（後述）。カーネルの複数バージョンごとに個別アドバイザリが発行されているため件数が多くなっています。実質の脆弱性は「PHP 系 2 件（Critical）」と「カーネル系 2 件（Important）」の計 4 CVE に集約されます。

---

## Critical（最優先）

今週の Critical は 4 件すべて **PHP** で、対象 CVE は共通です。バージョン系列ごとに分かれています。

| ALAS ID | パッケージ | 修正バージョン |
|---|---|---|
| [ALAS2023-2026-2041](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-2041.html) | php8.5 | 8.5.9 |
| [ALAS2023-2026-2042](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-2042.html) | php8.4 | 8.4.24 |
| [ALAS2023-2026-2043](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-2043.html) | php8.2 | 8.2.33 |
| [ALAS2023-2026-2044](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-2044.html) | php8.3 | 8.3.33 |

**関連 CVE:**

- **CVE-2026-17543（SQL インジェクション）** — 攻撃者が渡すパラメータ内のバックスラッシュ（`\`）のエスケープ（特殊文字の無害化処理）が不十分で、簡単に SQL インジェクション（不正な SQL 文を注入してデータベースを操作する攻撃）が成立し得る。影響範囲は PHP 8.2.x < 8.2.33 / 8.3.x < 8.3.33 / 8.4.x < 8.4.24 / 8.5.x < 8.5.9。DB を扱う PHP アプリでは影響が大きく、最優先で対処すべき。
- **CVE-2026-7260（DoS）** — phar アーカイブ（PHP の実行可能アーカイブ形式）内に循環するシンボリックリンク（互いを指し合うリンク）があると、無限再帰で C スタックを使い尽くし、PHP プロセスがクラッシュする。サービス停止（DoS＝サービス拒否）につながる。影響範囲は上記と同じ。

> PHP を稼働しているホストでは、上記の修正バージョン以上へ速やかに更新することを推奨します。特に CVE-2026-17543 は SQL インジェクションで実害が大きいため注意。

---

## Important（カーネルライブパッチ）

新規 Important は **80 件**、すべて `kernel-livepatch-*`（多数のカーネルバージョン向け）。対象 CVE は全件共通で以下の 2 件です。

- **CVE-2026-64531** — net/openvswitch: ネストした action 属性が過大なサイズの場合に拒否するよう修正（Open vSwitch＝仮想スイッチのネットワーク処理に関するカーネル修正）。
- **CVE-2026-64600** — xfs: ILOCK（inode ロック）を再取得した後にデータフォークのマッピングを再サンプリングするよう修正（XFS ファイルシステムの整合性に関するカーネル修正）。

代表例（ID はバージョン違いで連番。詳細取得できた先頭分を掲載）:

- [ALAS2023LIVEPATCH-2026-287](https://alas.aws.amazon.com/AL2023/ALAS2023LIVEPATCH-2026-287.html) — kernel-livepatch-6.18.38-73.137
- [ALAS2023LIVEPATCH-2026-288](https://alas.aws.amazon.com/AL2023/ALAS2023LIVEPATCH-2026-288.html) — kernel-livepatch-6.18.36-69.136
- [ALAS2023LIVEPATCH-2026-297](https://alas.aws.amazon.com/AL2023/ALAS2023LIVEPATCH-2026-297.html) — kernel-livepatch-6.1.176-221.367
- … 他 77 件（対象カーネルバージョンごとに発行）

> 実行中のカーネルに該当するライブパッチのみ適用されます。件数は多く見えますが、実体は上記 2 CVE のカーネル修正を各カーネルバージョンへ展開したものです。

> **注記:** 今回の Critical/Important は合計 84 件あり、フィード順で詳細取得したのは先頭 30 件（すべて上記ライブパッチ）でした。Critical 4 件（PHP）はスナップショット差分から別途特定して本記事に掲載しています。

---

## Medium（参考・新規のみ）

新規 Medium は 5 件（詳細は各リンク参照）:

- [ALAS2023-2026-1992](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1992.html) — libxml2
- [ALAS2023-2026-2025](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-2025.html) — amazon-cloudwatch-agent
- [ALAS2023-2026-2027](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-2027.html) — jbig2dec
- [ALAS2023-2026-2031](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-2031.html) — openssh
- [ALAS2023-2026-2038](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-2038.html) — openvpn

新規 Low: 0 件。

---

## 対処方法（参考・実行は管理者）

以下は AL2023 公式のアップデート手順です。**いずれも sudo（管理者権限）が必要**で、本監視タスクでは実行しません。実際の適用は管理者が判断・実施してください。

```bash
# リリース更新の有無を確認
sudo dnf check-release-update

# 更新可能なパッケージを確認
sudo dnf check-update

# セキュリティ更新のみ適用
sudo dnf upgrade --security
```

- 特定アドバイザリのみ適用する場合は `sudo dnf update --advisory <ALAS ID>` も利用できます。
- PHP の Critical 対応例: `sudo dnf update php8.4`（利用中の系列に読み替え）。

出典: [Amazon Linux 2023 ユーザーガイド — リポジトリと OS 更新の管理](https://docs.aws.amazon.com/ja_jp/linux/al2023/ug/managing-repos-os-updates.html)

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
