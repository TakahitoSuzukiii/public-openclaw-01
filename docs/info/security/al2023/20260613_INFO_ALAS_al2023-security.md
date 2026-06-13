# AL2023 セキュリティアドバイザリ週次まとめ（2026-06-13）

作成日: 2026-06-13 / STATUS: INFO / TOPIC: ALAS / 対象: Critical+Important

> ALAS（Amazon Linux Security Advisory）＝ Amazon Linux 公式のセキュリティ勧告。本記事は AWS 公式 ALAS フィードを基に、前回チェック（2026-06-08）以降に新規追加された勧告のうち、重大度が **Critical / Important** のものを日本語でまとめたものです。

## 概要

今回の新規アドバイザリは **2 件**。重大度別の内訳は次のとおりです。

- Critical: 0 件
- Important: 1 件
- Medium: 1 件
- Low: 0 件

本記事では Important の 1 件（docker）を詳しく扱います。Medium 1 件は末尾に簡潔に記載します。

---

## ALAS2023-2026-1835 — docker（Important）

- **ALAS ID:** [ALAS2023-2026-1835](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1835.html)
- **重大度:** Important（重要）
- **対象パッケージ:** docker（修正バージョンは上記 ALAS ページの「Affected Packages / New Packages」欄を参照のうえ、`dnf` で適用してください）
- **公開日:** 2026-06-12
- **関連 CVE:** CVE-2026-25680, CVE-2026-25681, CVE-2026-27136, CVE-2026-39821, CVE-2026-42502, CVE-2026-42506

### 内容の要約

docker（コンテナ実行エンジン）に同梱される Go 言語の HTML/ネットワーク処理ライブラリ群に、複数の脆弱性が見つかりました。主な内容は次のとおりです。

- **HTML 解析による過剰な CPU 消費（CVE-2026-25680）:** 任意の HTML を解析する際に CPU 時間を過剰に消費し、サービス拒否（DoS＝Denial of Service、正常なサービス提供を妨げる攻撃）につながる恐れがあります。
- **HTML サニタイズ回避による XSS（CVE-2026-25681 / CVE-2026-27136）:** 任意の HTML を解析・レンダリングすると、想定外の HTML ツリーが生成されることがあります。これを悪用すると、入力 HTML をサニタイズ（無害化）してから表示するアプリケーションでも XSS（Cross-Site Scripting、クロスサイトスクリプティング＝悪意あるスクリプトを他者のブラウザで実行させる攻撃）を成立させられる可能性があります。
- **Punycode/IDNA 変換の不備（CVE-2026-39821 ほか）:** ドメイン名変換を行う `ToASCII` / `ToUnicode` 関数が、本来 ASCII のみのラベルにデコードされる Punycode（国際化ドメイン名のエンコード方式）を誤って受理してしまう問題が含まれます。残る CVE（CVE-2026-42502 / CVE-2026-42506）も同系統の Go ライブラリに起因するものです。

いずれも単体で即時のシステム乗っ取りに至るものではありませんが、docker 経由で外部入力（HTML やドメイン名）を扱う構成では影響を受け得るため、早めの更新を推奨します。

---

## 対処方法（参考・実行は管理者）

以下は AL2023 公式の更新手順です。**いずれも sudo（管理者権限）が必要なため、本タスク（NEXUS）では実行しません。** 管理者の判断で適用してください。

```bash
# 適用可能なリリース更新の有無を確認
sudo dnf check-release-update

# 更新可能なパッケージを確認
sudo dnf check-update

# セキュリティ更新のみを適用
sudo dnf upgrade --security
```

出典: [Amazon Linux 2023 ユーザーガイド — リポジトリとOS更新の管理](https://docs.aws.amazon.com/ja_jp/linux/al2023/ug/managing-repos-os-updates.html)

---

## 新規 Medium（参考）

- ALAS2023-2026-1827 — mariadb114（Medium）

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
