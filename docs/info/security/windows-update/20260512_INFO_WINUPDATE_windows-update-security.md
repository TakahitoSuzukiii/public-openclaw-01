# Windows Update / Microsoft 月次セキュリティ更新まとめ（May 2026 Security Updates）

作成日: 2026-06-07 / 対象月: May 2026 Security Updates / STATUS: INFO / TOPIC: WINUPDATE / 出典: MSRC Security Update Guide(CVRF API)

## はじめに

本記事は Microsoft が毎月公開するセキュリティ更新（通称 **Patch Tuesday**＝米国時間の毎月第2火曜に公開される定例更新）のうち、**May 2026 Security Updates** をまとめたものです。

注意してほしいのは、**CVSS（Common Vulnerability Scoring System＝共通脆弱性評価システム。深刻度を 0〜10 の数値で表す指標）の基本値は「参考値」にすぎない**という点です。数値が高い＝必ず最優先で対応すべき、とは限りません。そこで本記事では Microsoft 公式のシグナルに沿って、次の優先度で整理しています。

1. **悪用が確認された脆弱性（ゼロデイ）** … すでに攻撃に使われているもの。最優先。
2. **公開済み（Publicly Disclosed）** … 詳細が一般に公開されており、攻撃者が悪用しやすい状態のもの。
3. **悪用される可能性が高い（Exploitation More Likely）** … Microsoft の Exploitability Index（悪用可能性指標）が「悪用されやすい」と判定したもの。
4. **要・更新適用かつ高深刻度（CVSS 9.8 以上の Windows/オンプレ Microsoft 製品）** … 利用者・管理者が自分で更新を当てる必要があり、かつ深刻度が極めて高いもの。

### 用語の補足

- **CVE（Common Vulnerabilities and Exposures）**: 脆弱性に振られる世界共通の識別番号。
- **ゼロデイ（zero-day）**: 修正プログラムが行き渡る前に悪用される脆弱性。
- **RCE（Remote Code Execution＝遠隔コード実行）**: 攻撃者が遠隔から任意のプログラムを実行できる、最も危険な部類の脆弱性。
- **EoP（Elevation of Privilege＝権限昇格）**: 一般ユーザー権限から管理者権限などへ不正に昇格する脆弱性。
- **Spoofing（なりすまし）**: 正規の相手・送信元になりすます脆弱性。
- **SFB（Security Feature Bypass＝セキュリティ機能のバイパス）**: 保護機構を回避する脆弱性。
- **DoS（Denial of Service＝サービス妨害）**: 対象を停止・無応答にする攻撃。
- **Exploitability Index（悪用可能性指標）**: Microsoft が「どれくらい悪用されやすいか」を判定した公式指標（Exploitation Detected / More Likely / Less Likely / Unlikely）。
- **KB番号**: 更新プログラムを識別する番号（Knowledge Base）。

## 概要

- **総CVE件数: 1,115 件**
- 重大度内訳: **Critical（緊急）61 / Important（重要）232 / Moderate（警告）396 / Low（低）30**
- ※総件数 1,115 には、重大度が付与されていない（Unknown）ものが **396 件** 含まれます。これは主に **Azure Linux（Mariner）など OSS 系**の項目で、一般的な Windows 利用者の対象外です。実質的に注目すべきは Critical/Important が中心です。

主要なシグナルのサマリ:

- 悪用確認（ゼロデイ）: **3 件**
- 公開済み（Publicly Disclosed）: **3 件**
- 悪用される可能性が高い（Exploitation More Likely）: **18 件**
- CVSS 9.8 以上で要・更新適用（Windows/オンプレ）: **4 件**
- CVSS 9.8 以上の Azure クラウド（利用者の対応不要・参考）: **9 件**
- CVSS 9.8 以上の Azure Linux（Mariner）/OSS: **9 件**（Windows 利用者の対象外・件数のみ）

## 【最優先】悪用が確認された脆弱性（ゼロデイ）

今月は **3 件** の悪用確認（Exploitation Detected）があります。すでに実際の攻撃で使われているため、最優先で更新を適用してください。

### 1. CVE-2026-42897 — Microsoft Exchange Server Spoofing Vulnerability
- 製品: Microsoft Exchange Server 2016 Cumulative Update 23
- 重大度: **Critical（緊急）** / 影響: Spoofing（なりすまし） / CVSS: 8.1
- KB: 5094144 / 5094142 / 5094140 / 5094139
- **なぜ危険か**: 自社で運用するオンプレミスの Exchange Server に対するなりすまし脆弱性で、すでに悪用が確認されています。メールサーバーは組織の中枢であり、なりすましが成立すると認証の偽装や情報窃取の足がかりになります。オンプレ Exchange を運用している場合は即時適用を強く推奨します。

### 2. CVE-2026-41091 — Microsoft Defender Elevation of Privilege Vulnerability
- 製品: Microsoft Malware Protection Engine（Defender のマルウェア対策エンジン）
- 重大度: **Important（重要）** / 影響: EoP（権限昇格） / CVSS: 7.8
- 公開済み（Publicly Disclosed）でもあり、悪用も確認済み。
- **なぜ危険か**: セキュリティ製品である Defender 自身の権限昇格脆弱性で、悪用が確認されています。攻撃者が端末上で管理者相当の権限を奪う足がかりになります。なお Defender のエンジンは通常**自動更新**されるため、多くの環境では特別な操作なく修正が適用されます。

### 3. CVE-2026-45498 — Microsoft Defender Denial of Service Vulnerability
- 製品: Microsoft Defender Antimalware Platform
- 重大度: **Low（低）** / 影響: DoS（サービス妨害） / CVSS: 4.0
- 公開済み（Publicly Disclosed）でもあり、悪用も確認済み。
- **なぜ危険か**: 深刻度自体は低いものの、Defender を停止・妨害させる悪用が確認されています。防御機能を無力化する前段として使われ得ます。こちらも Defender の自動更新で対応されるのが一般的です。

## 公開済み（Publicly Disclosed）

詳細が一般に公開されており、攻撃者が悪用に着手しやすい状態の脆弱性です。今月は **3 件**。

- **CVE-2026-41091** — Microsoft Defender Elevation of Privilege Vulnerability（Important / 上記ゼロデイと重複）
- **CVE-2026-45585** — Windows BitLocker Security Feature Bypass Vulnerability（Important / SFB / Exploitability Index は「悪用されやすい」判定。物理アクセス時に BitLocker のディスク暗号化保護を回避される恐れ）
- **CVE-2026-45498** — Microsoft Defender Denial of Service Vulnerability（Low / 上記ゼロデイと重複）

## 悪用される可能性が高い（Exploitation More Likely）

以下は Microsoft 公式の **Exploitability Index** が「悪用されやすい」と判定した脆弱性です（全 **18 件**）。CVSS 値の大小ではなく、この公式判定に基づく注意リストです。重大度の高いものを中心に主要なものを挙げます。

- **CVE-2026-42822**（Critical / EoP / CVSS 10）Azure Local Disconnected Operations (ALDO) — Azure Local 製品。後述の「要更新適用」にも該当。
- **CVE-2026-41103**（Critical / EoP / CVSS 9.1）Microsoft SSO Plugin for Jira & Confluence
- **CVE-2026-35435**（Critical / EoP / CVSS 8.6）Azure AI Foundry（クラウド側・利用者対応不要）
- **CVE-2026-40364**（Critical / RCE / CVSS 8.4）Microsoft Word — Office 2019 ほか。文書を開く操作で悪用され得る。KB:5002858
- **CVE-2026-40361**（Critical / RCE / CVSS 8.4）Microsoft Outlook and Word — Office 2019 ほか。KB:5002858
- **CVE-2026-45495**（Important / RCE / CVSS 8.8）Microsoft Edge (Chromium-based)
- **CVE-2026-33840 / CVE-2026-35417**（Important / EoP / CVSS 7.8）Win32k（Windows のカーネル系コンポーネント）
- **CVE-2026-33835**（Important / EoP / CVSS 7.8）Windows Cloud Files Mini Filter Driver
- **CVE-2026-33837**（Important / EoP / CVSS 7.8）Windows TCP/IP
- **CVE-2026-40369 / CVE-2026-33841**（Important / EoP / CVSS 7.8）Windows Kernel
- **CVE-2026-40397**（Important / EoP / CVSS 7.8）Windows Common Log File System Driver（CLFS。過去にゼロデイ多発の要注意ドライバ）
- **CVE-2026-40398**（Important / EoP / CVSS 7.8）Windows Remote Desktop Services
- **CVE-2026-35416**（Important / EoP / CVSS 7.0）Windows Ancillary Function Driver for WinSock（AFD.sys）
- **CVE-2026-45585**（Important / SFB）Windows BitLocker（公開済みにも該当）
- **CVE-2026-45494 / CVE-2026-45492**（Moderate）Microsoft Edge (Chromium-based)

> Win32k / Kernel / CLFS / AFD.sys といった OS 中核の権限昇格は、別の侵入経路と組み合わされて「管理者権限の奪取」に使われやすい定番です。Windows 本体の月例更新を確実に当ててください。

## 【特に注意・優先的に更新適用すべき】高深刻度（CVSS 9.8 以上）で利用者の更新適用が必要なもの

ここが本記事の中核です。**CVSS は参考値**ですが、以下は **Microsoft 製品（Windows/オンプレ）として高深刻度（CVSS 9.8 以上）かつ利用者・管理者による更新適用が必要**なため、優先対応を推奨します。全 **4 件**。

### 1. CVE-2026-42822 — Azure Local Disconnected Operations (ALDO) Elevation of Privilege Vulnerability
- 製品: Azure Local（旧称 Azure Stack HCI。オンプレで動かす Azure 基盤）
- 重大度: **Critical** / 影響: EoP（権限昇格） / **CVSS: 10.0（最大値）**
- Exploitability Index: **Exploitation More Likely（悪用されやすい）**
- KB: （CVRF に個別 KB の記載なし。Azure Local の更新チャネル経由で適用）
- 4 件の中で唯一「悪用されやすい」判定であり、CVSS も満点。Azure Local を運用している場合は最優先で対応してください。

### 2. CVE-2026-42898 — Microsoft Dynamics 365 On-Premises Remote Code Execution Vulnerability
- 製品: Microsoft Dynamics 365 (on-premises) version 9.1
- 重大度: **Critical** / 影響: RCE（遠隔コード実行） / **CVSS: 9.9**
- Exploitability Index: Exploitation Unlikely（悪用されにくい）
- KB: 5078943
- オンプレ Dynamics 365 を運用している組織は適用必須。RCE のため放置すればサーバー乗っ取りに直結します。

### 3. CVE-2026-41089 — Windows Netlogon Remote Code Execution Vulnerability
- 製品: Windows Server 2019（ほか各 Windows Server）
- 重大度: **Critical** / 影響: RCE / **CVSS: 9.8**
- Exploitability Index: Exploitation Less Likely（悪用されにくい）
- KB: 5087538 / 5087545 / 5087424 / 5087539 / 5087423 / 5087541 / 5087537 / 5087470 / 5087471
- Netlogon はドメイン認証の中核プロトコル。ドメインコントローラを運用している組織は特に優先して適用してください。

### 4. CVE-2026-41096 — Windows DNS Client Remote Code Execution Vulnerability
- 製品: Windows Server 2025 (Server Core installation)（ほか各 Windows）
- 重大度: **Critical** / 影響: RCE / **CVSS: 9.8**
- Exploitability Index: Exploitation Unlikely（悪用されにくい）
- KB: 5087539 / 5087423 / 5089549 / 5089466 / 5087420 / 5087541 / 5089548
- DNS クライアント側の RCE。名前解決の応答を悪用される経路で、クライアント・サーバー双方に影響します。月例更新で対応してください。

> 補足: 上記のうち CVE-2026-41089 / CVE-2026-41096 は Exploitability Index が「悪用されにくい」判定ですが、**RCE かつ CVSS 9.8 以上**という深刻度の高さから、優先適用の対象として挙げています。

## 参考: Azure クラウドの高スコア（利用者の更新適用は不要）

以下は CVSS 9.8 以上ですが、**Azure クラウド側で Microsoft がすでに修正済み**であり、**利用者の操作は原則不要**です（全 9 件）。参考情報として一覧化します。

- CVE-2026-40412（CVSS 10）Azure Orbital Spatio — RCE
- CVE-2026-23652（CVSS 10）Microsoft Power Pages — RCE
- CVE-2026-47280（CVSS 10）Azure Resource Manager — EoP
- CVE-2026-42826（CVSS 10）Azure DevOps — Information Disclosure（情報漏えい）
- CVE-2026-41104（CVSS 10）Microsoft Planetary Computer Pro — 情報漏えい
- CVE-2026-42901（CVSS 10）Microsoft Entra ID — EoP
- CVE-2026-33109（CVSS 9.9）Azure Managed Instance for Apache Cassandra — RCE
- CVE-2026-40411（CVSS 9.9）Azure Virtual Network Gateway — RCE
- CVE-2026-42823（CVSS 9.9）Azure Logic Apps — EoP

## 参考: Azure Linux（Mariner）/ OSS

CVSS 9.8 以上の Azure Linux（Mariner）/ OSS 関連は **9 件**（件数のみ）。これらは **Windows 利用者の対象外**です。

## 影響タイプ別の件数（byImpact）

| 影響タイプ | 件数 |
|---|---|
| Elevation of Privilege（権限昇格） | 70 |
| Remote Code Execution（遠隔コード実行） | 38 |
| Information Disclosure（情報漏えい） | 19 |
| Spoofing（なりすまし） | 16 |
| Denial of Service（サービス妨害） | 9 |
| Security Feature Bypass（機能バイパス） | 8 |
| Tampering（改ざん） | 3 |

※上記のほか Unknown（重大度・影響未付与）が 952 件あり、その大半は Azure Linux（Mariner）/OSS 系で Windows 利用者の対象外です。

## 対処方法（適用は利用者／管理者が実施）

更新の適用はクラウド側で完結する Azure 系を除き、**利用者・管理者の側で実施する必要があります**。

- **個人・一般の Windows**: 「設定 → Windows Update → 更新プログラムのチェック」から最新の更新を適用し、再起動してください。
- **企業・組織**: WSUS（Windows Server Update Services）や Microsoft Intune を使い、対象 KB を検証のうえ計画的に配布してください。
- **オンプレ製品（Exchange / Dynamics 365 / Azure Local 等）**: 各製品の更新チャネル・累積更新（CU）を適用してください。本記事のゼロデイ・高深刻度項目は特に優先。
- **Microsoft Defender / Edge**: 通常は自動更新されますが、念のため定義・バージョンが最新か確認してください。
- Microsoft の月次更新は **毎月第2火曜（米国時間、Patch Tuesday）** に公開されます。

### 出典

- MSRC ブログ（当月サマリ）: https://www.microsoft.com/en-us/msrc/blog/2026/05/202605-security-update
- MSRC ブログ（トップ）: https://www.microsoft.com/en-us/msrc/blog/
- Security Update Guide: https://msrc.microsoft.com/update-guide/

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
