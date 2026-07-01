# プラットフォーム診断 入門シリーズ（インデックス）

> 作成日 2026-06-16 ／ 自社ITサービスのインフラ・セキュリティエンジニア向けの学習ノート。
> 専門用語は「綴り＋意味」をセットで補足し、平易に解説する方針。図は Mermaid 埋め込み（GitHub でネイティブ描画）。

「プラットフォーム診断とは何か」を、ソースコード診断・脆弱性診断・OWASP・SAST/DAST などの体系の中で理解し、製品（特に Tenable）とベスト/バッドプラクティスまで一気通貫でまとめたシリーズ。

| # | 記事 | 内容 |
|---|---|---|
| 1 | [全体像と体系](20260616_INFO_SECURITY_platform-assessment-01-basics.md) | アプリ層/インフラ層の切り分け、プラットフォーム診断の定義・対象・観点・目的とリスク、Webアプリ診断との違い、SAST/DAST/IAST/SCA/OWASP との位置づけ |
| 2 | [主要ツールと Tenable 深掘り](20260616_INFO_SECURITY_platform-assessment-02-tools-and-tenable.md) | 定番スキャナ/VMプラットフォーム比較、Tenable 製品ライン（Nessus／VM／Security Center／One）、CVSS/EPSS/VPR、自社運用での使いどころ |
| 3 | [ベスト/バッドプラクティス](20260616_INFO_SECURITY_platform-assessment-03-bestpractices.md) | プラットフォーム診断と Tenable 運用のグッド/アンチパターン、診断とペネトレの違い、運用サイクル |

## 要点（30秒サマリ）

- **プラットフォーム診断**＝OS・ミドルウェア・NW機器・クラウド基盤など**インフラ層**の既知脆弱性・設定不備を洗い出す検査（別名ネットワーク診断）。**Webアプリ診断と補完関係**。
- アプリ層には **SAST/DAST/IAST/SCA**、判断基準に **OWASP（Top10/ASVS）**。**1ツールで網羅は不可能**、多層＋手動診断の組み合わせが現実解。
- ツール定番は **Nessus / Tenable VM・Security Center / Qualys / Rapid7 / Defender / OpenVAS**。**Tenable** は Nessus を中核に **CVSS+EPSS+VPR** で現実的な優先度づけが強み。
- 成否は **「検出→是正→再確認のループを回し切るか」**。レポートを出して終わりにしない。

> 注: 公開情報＋一般的な実務知識に基づく解説。製品仕様・名称・指標は更新が早いため、導入検討時は各社公式で要確認。
