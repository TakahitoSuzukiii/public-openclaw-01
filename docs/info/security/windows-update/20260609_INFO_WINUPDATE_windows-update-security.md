# Windows / Microsoft 月次セキュリティ更新まとめ — June 2026 Security Updates

作成日: 2026-06-10 / 対象月: June 2026 Security Updates / STATUS: INFO / TOPIC: WINUPDATE / 出典: MSRC Security Update Guide(CVRF API)

## はじめに

本記事は Microsoft の月次セキュリティ更新（Patch Tuesday）の内容を、初心者にも分かるようにまとめたものです。

注意していただきたいのは、**CVSS（Common Vulnerability Scoring System=共通脆弱性評価システム）の基本値は「参考値」**であり、それだけを見て優先度を決めるべきではない、という点です。本記事は MSRC（Microsoft Security Response Center=マイクロソフトのセキュリティ対応チーム）が公式に示すシグナルに基づき、**「悪用確認 → 公開済み → 悪用される可能性が高い → 要・更新適用の高深刻度」**という優先度で整理しています。

**用語の注釈:**
- **RCE**（Remote Code Execution=遠隔コード実行）: 攻撃者がネットワーク越しに任意のプログラムを実行できる、最も危険な部類の脆弱性。
- **EoP**（Elevation of Privilege=権限昇格）: 一般ユーザ権限から管理者権限などへ不正に昇格される脆弱性。
- **ゼロデイ**: 修正プログラムが出る前に、すでに攻撃に悪用されている脆弱性。
- **Exploitability Index**（悪用可能性指標）: Microsoft が「この脆弱性がどれくらい悪用されやすいか」を独自評価した指標。`Exploitation More Likely`（悪用される可能性が高い）が最も警戒すべき区分。
- **KB**（Knowledge Base 番号）: 各更新プログラムに振られた識別番号。

---

## 概要

- **全 CVE 件数: 606 件**
  - Critical（緊急）: 39 件
  - Important（重要）: 184 件
  - Moderate（警告）: 18 件
  - Low（注意）: 5 件
  - Unknown（重大度未付与）: 360 件

※ 全 606 件のうち 360 件は重大度が「未付与（Unknown）」です。これは Azure Linux（Mariner）など、Windows 利用者の直接の対象ではない OSS 由来の項目が多数含まれているためで、Windows ユーザが個別に対応すべき件数はこれより大幅に少なくなります。

---

## 【最優先】悪用が確認された脆弱性（ゼロデイ）

**今回、悪用が確認された（ゼロデイの）脆弱性は 0 件です。** 現時点で実際の攻撃に使われている報告のある脆弱性はありません。とはいえ油断は禁物で、下記の「公開済み」「悪用される可能性が高い」項目は早期の更新適用が推奨されます。

---

## 公開済み（Publicly Disclosed）— 3 件

修正前に脆弱性の詳細が一般に公開されていたもの。攻撃者が情報を入手しやすいため注意が必要です。

- **CVE-2026-45586**（Important / EoP）: Windows Collaborative Translation Framework (CTFMON) Elevation of Privilege Vulnerability — CVSS 7.8。悪用可能性は「More Likely（高い）」。
- **CVE-2026-49160**（Important / DoS=Denial of Service=サービス妨害）: HTTP.sys Denial of Service Vulnerability — CVSS 7.5。悪用可能性は「More Likely」。
- **CVE-2026-50507**（Important / Security Feature Bypass=セキュリティ機能のバイパス）: Windows BitLocker Security Feature Bypass Vulnerability — CVSS 6.8。悪用可能性は「More Likely」。物理アクセスを要する区分です。

---

## 悪用される可能性が高い（Exploitation More Likely）— 15 件

以下は Microsoft 公式の **Exploitability Index** で「悪用される可能性が高い」と評価された脆弱性です。CVSS 値の大小に関わらず、Microsoft が「狙われやすい」と判断したものなので、優先的に更新を適用してください。重大度の高いものを中心に主要なものを挙げます。

**Critical（緊急）:**
- **CVE-2026-47291** — HTTP.sys Remote Code Execution Vulnerability（RCE / CVSS 9.8）
- **CVE-2026-42985** — Remote Desktop Client Remote Code Execution Vulnerability（RCE / CVSS 8.8）
- **CVE-2026-44803** — Windows Graphics Component Remote Code Execution Vulnerability（RCE / CVSS 7.8）
- **CVE-2026-44812** — Windows Graphics Component Remote Code Execution Vulnerability（RCE / CVSS 7.8）

**Important（重要）抜粋:**
- **CVE-2026-45586** — CTFMON EoP（CVSS 7.8、公開済み）
- **CVE-2026-45658** — Windows BitLocker Security Feature Bypass（CVSS 7.8）
- **CVE-2026-42905** — Windows DWM Core Library EoP（CVSS 7.8）
- **CVE-2026-42980** — NT OS Kernel EoP（CVSS 7.8）
- **CVE-2026-42986** — Microsoft Graphics Component EoP（CVSS 7.8）
- **CVE-2026-42989** — Winlogon EoP（CVSS 7.8）
- **CVE-2026-49160** — HTTP.sys DoS（CVSS 7.5、公開済み）
- **CVE-2026-47634 / CVE-2026-45481** — Microsoft SharePoint Server Spoofing（なりすまし / CVSS 7.3）
- **CVE-2026-50507** — Windows BitLocker Security Feature Bypass（CVSS 6.8、公開済み）
- **CVE-2026-50508** — Windows NTLM Spoofing（CVSS 6.5）

---

## 【特に注意・優先的に更新適用すべき】高深刻度（CVSS≧9.8）で更新適用が必要なもの — 5 件

以下は Microsoft 製品として高深刻度（CVSS 9.8 以上）かつ、**利用者・管理者による更新適用が必要**な脆弱性です。CVSS は参考値ですが、これらは深刻度が高く更新が必要なため、優先対応を強く推奨します。

### 1. CVE-2026-45657 — Windows Kernel Remote Code Execution Vulnerability
- 製品: Windows Server 2022 ほか / 重大度: **Critical** / 影響: RCE / CVSS: **9.8**
- Exploitability Index: Exploitation Less Likely（悪用される可能性は低い）
- KB: 5094128 / 5094125 / 5094126 / 5093998 / 5095051
- Windows カーネル（OS の中核）に対する遠隔コード実行。成立すれば最高権限での侵害につながり得ます。

### 2. CVE-2026-47291 — HTTP.sys Remote Code Execution Vulnerability
- 製品: Windows 10/11、Windows Server 系 / 重大度: **Critical** / 影響: RCE / CVSS: **9.8**
- Exploitability Index: **Exploitation More Likely（悪用される可能性が高い）**
- KB: 5094123 / 5094128 / 5094127 / 5094125 / 5094126 / 5093998 / 5095051 / 5094122 / 5094042 / 5094041
- HTTP.sys（Windows の HTTP 通信を処理するカーネルドライバ）の RCE。Web サーバ（IIS 等）を公開している環境では特に危険で、本月で**最も警戒すべき1件**です。

### 3. CVE-2026-26142 — Nuance PowerScribe Remote Code Execution Vulnerability
- 製品: Nuance PowerScribe 360 version 4.0.5 / 重大度: **Critical** / 影響: RCE / CVSS: **9.8**
- Exploitability Index: Exploitation Less Likely
- KB: （個別 KB なし。製品側の更新手順に従ってください）
- 医療向け音声認識製品 PowerScribe の RCE。該当製品の利用者は提供元の更新案内に従ってください。

### 4. CVE-2026-44815 — DHCP Client Service Remote Code Execution Vulnerability
- 製品: Windows 10/11、Windows Server 系 / 重大度: **Critical** / 影響: RCE / CVSS: **9.8**
- Exploitability Index: Exploitation Less Likely
- KB: 5094123 / 5094128 / 5094127 / 5094125 / 5094126 / 5093998 / 5095051 / 5094122 / 5094042 / 5094041
- DHCP クライアント（IP アドレス自動取得の仕組み）の RCE。悪意ある DHCP 応答で攻撃され得るため、同一ネットワーク内の脅威に注意が必要です。

### 5. CVE-2026-47643 — Azure Stack Edge Remote Code Execution Vulnerability
- 製品: Azure Stack Edge / 重大度: **Important** / 影響: RCE / CVSS: **9.8**
- Exploitability Index: Exploitation Unlikely（悪用は考えにくい）
- KB: （製品側の更新手順に従ってください）
- オンプレ設置型のエッジ機器 Azure Stack Edge の RCE。該当機器の運用者は更新を確認してください。

---

## 参考: Azure クラウドの高スコア（利用者の更新適用は不要）— 1 件

- **CVE-2026-48567** — Azure HorizonDB Elevation of Privilege Vulnerability（Critical / CVSS **10.0**）

これは Azure クラウド側で **Microsoft がすでに修正済み**であり、利用者側の操作は原則不要です。CVSS は満点の 10.0 ですが、クラウドサービスとして自動的に保護されるため、参考扱いとします。

## 参考: Azure Linux（Mariner）/ OSS

CVSS 9.8 以上の Azure Linux（Mariner）/ OSS 関連は **0 件**でした。これらは Windows 利用者の直接の対象外です。

---

## 影響タイプ別の件数（byImpact）

| 影響タイプ | 件数 |
|---|---|
| Elevation of Privilege（権限昇格） | 67 |
| Remote Code Execution（遠隔コード実行） | 55 |
| Information Disclosure（情報漏えい） | 30 |
| Spoofing（なりすまし） | 27 |
| Security Feature Bypass（機能バイパス） | 19 |
| Denial of Service（サービス妨害） | 7 |
| Tampering（改ざん） | 3 |

※ Unknown（影響タイプ未分類）398 件は OSS 由来等を多く含むため除外しています。

---

## 対処方法（適用は利用者・管理者が行います）

- **個人/一般の Windows ユーザ:** 「設定」→「Windows Update」→「更新プログラムのチェック」から最新の更新を適用してください。再起動が必要な場合があります。
- **企業/組織の管理者:** WSUS（Windows Server Update Services）や Microsoft Intune を用いて、対象端末・サーバへ計画的に展開してください。公開サーバ（IIS 等）を持つ環境は、特に HTTP.sys 関連（CVE-2026-47291）を優先適用してください。
- **補足:** Microsoft の月次セキュリティ更新は毎月**第2火曜（米国時間, Patch Tuesday）**に公開されます。

**出典:**
- MSRC ブログ 当月サマリ: https://www.microsoft.com/en-us/msrc/blog/2026/06/202606-security-update
- MSRC ブログ トップ: https://www.microsoft.com/en-us/msrc/blog/
- Security Update Guide: https://msrc.microsoft.com/update-guide/

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
