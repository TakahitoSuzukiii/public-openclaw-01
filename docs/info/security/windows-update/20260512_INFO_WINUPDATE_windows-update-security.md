# Windows Update / Microsoft 月次セキュリティ更新まとめ — May 2026 Security Updates

作成日: 2026-06-07 / 対象月: May 2026 Security Updates / STATUS: INFO / TOPIC: WINUPDATE / 出典: MSRC Security Update Guide(CVRF API)

---

## この記事について

Microsoft（マイクロソフト）は毎月「Patch Tuesday（パッチ・チューズデー）」と呼ばれる定例日に、自社製品のセキュリティ修正をまとめて公開します。米国時間の毎月第2火曜日が公開日です。このまとめは、MSRC（Microsoft Security Response Center=マイクロソフトのセキュリティ対応センター）が公式に提供する機械可読 API（CVRF=Common Vulnerability Reporting Framework）から取得したデータをもとに、初心者の方にも分かるよう日本語で要点を整理したものです。

専門用語は最初に綴りと意味をセットで補足します。

- **CVE（Common Vulnerabilities and Exposures）**＝脆弱性に振られる世界共通の識別番号。
- **CVSS（Common Vulnerability Scoring System）**＝脆弱性の深刻さを 0〜10 の数値で表す指標。10 に近いほど危険。
- **RCE（Remote Code Execution）**＝遠隔コード実行。攻撃者が離れた場所から任意のプログラムを実行できてしまう、最も危険な部類の欠陥。
- **EoP（Elevation of Privilege）**＝権限昇格。一般ユーザー権限から管理者権限などへ不正に昇格される欠陥。
- **ゼロデイ（Zero-day）**＝修正が出る前に、すでに実際の攻撃に悪用されている脆弱性。最優先で対処すべきもの。

---

## 概要

今月（May 2026）のセキュリティ更新では、合計 **1,115 件** の CVE が公開されました。このうち重大度（深刻さのランク）が付与されているものの内訳は次のとおりです。

- **Critical（緊急）: 61 件**
- **Important（重要）: 232 件**
- **Moderate（警告）: 396 件**
- **Low（低）: 30 件**

注意点として、全体 1,115 件のうち **396 件は重大度が「未付与（Unknown）」** です。これは Azure Linux（旧称 Mariner、Microsoft が提供する Linux ディストリビューション）に含まれるオープンソース由来のパッケージ更新などが多数含まれているためで、Microsoft 製品としての重大度ラベルが付いていないものです。一般の Windows 利用者・管理者がまず注目すべきなのは、Microsoft 製品として重大度が付与された脆弱性（特に Critical と、悪用が確認されたもの）です。

---

## 【最重要】悪用が確認された脆弱性（ゼロデイ）

今月は **3 件** の脆弱性で、すでに実際の攻撃に悪用されている（Exploitation Detected）ことが確認されています。これらは最優先で更新を適用すべきものです。

### 1. CVE-2026-42897 — Microsoft Exchange Server Spoofing Vulnerability
- 重大度: **Critical（緊急）** / 影響タイプ: Spoofing（なりすまし） / CVSS: **8.1**
- Exchange Server（オンプレミスのメールサーバ製品）のなりすましの脆弱性です。攻撃者が正規の相手や正規の通信を装える恐れがあり、メールサーバは組織の機密情報の中枢であるため影響が大きいです。すでに悪用が確認されているため、Exchange を運用している組織は最優先でパッチを適用してください。

### 2. CVE-2026-41091 — Microsoft Defender Elevation of Privilege Vulnerability
- 重大度: **Important（重要）** / 影響タイプ: EoP（権限昇格） / CVSS: **7.8**
- Windows 標準のセキュリティ対策製品 Microsoft Defender における権限昇格の脆弱性です。悪用されると、攻撃者が一般権限から高い権限を奪取できます。悪用確認に加え一般公開（Publicly Disclosed）もされており、危険度が高いため早急な適用が必要です。

### 3. CVE-2026-45498 — Microsoft Defender Denial of Service Vulnerability
- 重大度: **Low（低）** / 影響タイプ: DoS（Denial of Service=サービス妨害） / CVSS: **4.0**
- 同じく Microsoft Defender における、サービスを停止・妨害させる脆弱性です。重大度ラベル自体は Low ですが、すでに悪用が確認されており、かつ一般公開もされているため油断はできません。Defender の保護が無効化されると他の攻撃の足がかりになり得ます。更新を適用してください。

---

## 一般公開（Publicly Disclosed）された脆弱性

修正前に脆弱性の情報が一般に公開されていたもの（攻撃者に利用されやすい状態だったもの）は **3 件** です。

- **CVE-2026-45498** — Microsoft Defender Denial of Service Vulnerability（重大度: Low / 上記ゼロデイと同一）
- **CVE-2026-41091** — Microsoft Defender Elevation of Privilege Vulnerability（重大度: Important / 上記ゼロデイと同一）
- **CVE-2026-45585** — Windows BitLocker Security Feature Bypass Vulnerability（重大度: Important）
  - BitLocker（Windows のディスク暗号化機能）のセキュリティ機能バイパスの脆弱性です。悪用確認はまだですが「Exploitation More Likely（悪用される可能性が高い）」と評価されており、物理的にアクセスできる端末で暗号化保護が回避される恐れがあります。

---

## 重大（Critical）な脆弱性の注目株

Critical 61 件の中から、CVSS が高く影響の大きいもの（特に RCE / EoP）を抜粋して紹介します。

| CVE | タイトル | 影響タイプ | CVSS |
|---|---|---|---|
| CVE-2026-40412 | Azure Orbital Spatio Remote Code Execution | RCE | 10.0 |
| CVE-2026-23652 | Microsoft Power Pages Remote Code Execution | RCE | 10.0 |
| CVE-2026-47280 | Azure Resource Manager Elevation of Privilege | EoP | 10.0 |
| CVE-2026-42822 | Azure Local Disconnected Operations (ALDO) Elevation of Privilege | EoP | 10.0 |
| CVE-2026-42901 | Microsoft Entra ID Elevation of Privilege | EoP | 10.0 |
| CVE-2026-33109 | Azure Managed Instance for Apache Cassandra RCE | RCE | 9.9 |
| CVE-2026-40411 | Azure Virtual Network Gateway Remote Code Execution | RCE | 9.9 |
| CVE-2026-42898 | Microsoft Dynamics 365 On-Premises Remote Code Execution | RCE | 9.9 |
| CVE-2026-41089 | Windows Netlogon Remote Code Execution | RCE | 9.8 |
| CVE-2026-41096 | Windows DNS Client Remote Code Execution | RCE | 9.8 |

補足: 一般の Windows 端末利用者にとって特に身近なのは、**Windows Netlogon（CVE-2026-41089）** や **Windows DNS Client（CVE-2026-41096）** といった Windows コンポーネントの RCE です。一方、CVSS 10.0 の上位の多くは Azure（Microsoft のクラウドサービス）関連で、主にクラウド側で対処されるものです。Office / Word / Outlook 系の RCE（CVE-2026-40363 / 40364 / 40361 ほか、CVSS 8.4）も多数あり、悪意ある文書ファイルを開くことで被害につながり得るため注意が必要です。

---

## 影響タイプ別の件数

重大度や種別ごとに、どんな種類の脆弱性が多かったかの内訳です（件数の多い順、Unknown は末尾）。

- Elevation of Privilege（権限昇格）: 70 件
- Remote Code Execution（遠隔コード実行）: 38 件
- Information Disclosure（情報漏えい）: 19 件
- Spoofing（なりすまし）: 16 件
- Denial of Service（サービス妨害）: 9 件
- Security Feature Bypass（セキュリティ機能の回避）: 8 件
- Tampering（改ざん）: 3 件
- Unknown（未分類 / 主に Azure Linux 等）: 952 件

---

## 対処方法（参考。実際の適用は利用者・管理者の判断で）

- **個人 / 一般の Windows 端末:** 「設定」→「Windows Update」→「更新プログラムのチェック」を実行し、提供された更新を適用して再起動してください。自動更新を有効にしておくのが基本です。
- **企業 / 組織環境:** WSUS（Windows Server Update Services）や Microsoft Intune を使った集中配布で、検証後に展開してください。特に今月は Exchange Server・Microsoft Defender のゼロデイがあるため、該当製品を運用している場合は優先的に対応を。
- **公開タイミング:** Microsoft の月次セキュリティ更新は、米国時間の毎月第2火曜日（Patch Tuesday）に公開されます。

### 出典・参考リンク

- MSRC ブログ（月次サマリ一覧）: https://www.microsoft.com/en-us/msrc/blog
- 当月サマリ（推定 URL）: https://www.microsoft.com/en-us/msrc/blog/2026/05/202605-security-update
- Security Update Guide（公式の脆弱性データベース）: https://msrc.microsoft.com/update-guide/

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
