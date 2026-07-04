# AL2023 セキュリティアドバイザリ 週次レポート（Critical / Important）

作成日: 2026-07-04 / STATUS: INFO / TOPIC: ALAS / 対象: Critical+Important

Amazon Linux 2023（AL2023）の公式セキュリティアドバイザリ **ALAS（Amazon Linux Security Advisory）** を週次で監視し、新規に公開された **Critical（緊急）** および **Important（重要）** を要約します。情報源は AWS 公式 ALAS RSS フィードのみです。

## 概要（今週の新規アドバイザリ）

今週（前回チェック 2026-06-27 → 今回 2026-07-04）の新規アドバイザリは **2 件** です。

| 重大度 | 件数 |
|---|---|
| Critical（緊急） | 1 |
| Important（重要） | 0 |
| Medium（中） | 1 |
| Low（低） | 0 |

本レポートで詳細を扱うのは Critical / Important の **1 件** です。

---

## Critical（緊急）

### ALAS2023-2026-1907 — rclone

- **アドバイザリ:** [ALAS2023-2026-1907](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1907.html)
- **重大度:** Critical（緊急）
- **対象パッケージ:** rclone（**修正バージョン: 1.74.3**）
- **関連 CVE:** [CVE-2026-49980](https://alas.aws.amazon.com/cve/html/CVE-2026-49980.html)

**要約:**
rclone（複数のクラウドストレージ間でファイル同期を行うコマンドラインツール）に、認証を回避した遠隔コード実行（RCE: Remote Code Execution、リモートからの任意コマンド実行）の脆弱性があります。バージョン 1.46.0 から 1.74.2 までの `rclone rcd --rc-serve` は、`/[remote:path]/object` 形式のパスに対して、**認証なしの GET / HEAD リクエストを受け付けてしまいます**。

URL から解析された remote 値は通常のバックエンド初期化処理に渡されますが、この際にインラインで指定された remote 設定はバックエンドのオプションを設定でき、初期化中にローカルコマンドを実行し得ます。結果として、**認証なしの GET / HEAD リクエスト 1 回だけで、rclone プロセスの実行ユーザ権限で任意のコマンドが実行**されてしまいます。この脆弱性は **1.74.3 で修正**されています。

`rclone rcd`（リモートコントロールデーモン）を `--rc-serve` 付きで公開している環境は特に注意が必要です。

---

## Medium / Low（新規・参考）

詳細対象外ですが、新規で以下が公開されています（参考）。

- **Medium 1 件:** [ALAS2023-2026-1906](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1906.html)（ecs-init）
- **Low:** 0 件

---

## 対処方法（参考・実行は管理者）

以下は AL2023 公式のアップデート手順です。**いずれも sudo（管理者権限）が必要なため、本監視タスクでは実行しません。** 適用は管理者の判断で行ってください。

```bash
# リリース更新の有無を確認
sudo dnf check-release-update

# 更新可能なパッケージを確認
sudo dnf check-update

# セキュリティ更新のみ適用
sudo dnf upgrade --security
```

出典: [AL2023 リポジトリと OS の更新の管理（AWS 公式）](https://docs.aws.amazon.com/ja_jp/linux/al2023/ug/managing-repos-os-updates.html)

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
