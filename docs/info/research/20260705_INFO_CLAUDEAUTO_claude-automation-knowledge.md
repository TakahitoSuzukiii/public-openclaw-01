# 週次まとめ：Claude / Claude Code / Cowork による業務自動化ナレッジ（2026-07-05 号）

作成日: 2026-07-05 / STATUS: INFO / TOPIC: CLAUDEAUTO

> 注: 各記事の主張・数値・手順は出典元のものです。製品仕様やコマンドは変わり得るため、導入時は必ず一次情報で再確認してください。本記事は要約・考察であり全文転載ではありません。専門用語は綴りと意味を併記します（例: E2E=End to End=利用者操作の端から端までを通しで検証するテスト）。

## 今週のテーマ・見どころ

前週まではインフラの IaC（Infrastructure as Code＝コードでインフラを定義・構築する手法）や SRE（Site Reliability Engineering＝サイト信頼性工学）に寄っていたので、今週は**「資料作成（Excel／PowerPoint）」と「非エンジニア向けの Cowork」**を主役に据えました。加えて、2026年2月に正式提供が始まった **Claude Code Security（脆弱性発見・修正）** の最新動向と、日本企業の **Playwright × Claude Code による E2E テスト自動化の実績**を取り上げます。「面倒なルーティンワーク（レポート作成）を丸ごと委譲する」という、鈴木さんの優先テーマに直結する回です。

---

## 1. ルーティンワーク自動化の本命 ── Excel 集計・PowerPoint 資料作成

インフラ運用でも避けて通れない「月次レポート・報告資料づくり」を、Claude で自動化する事例が一気に増えました。

### 1-1. Claude Cowork で広告月次レポートを自動化（CSV→Excel→PPT を一括）
- 出典URL: https://riddle-sense.co.jp/blog/claude-cowork-ad-report-automation/
- 「毎月 CSV をダウンロード → Excel で集計 → PowerPoint に転記」という半日仕事を、Cowork に自動化させた実例。手順は3ステップで、①要件を会話で伝えて Python スクリプトを生成させる、②それを「スキル」（指示と処理をセットにした自動化パッケージ）として登録、③2か月目以降は「レポート生成して」の一言で実行、という流れです。**"一度スキル化すれば以後はワンフレーズ"** という設計思想が、運用の定型作業にそのまま応用できます。

### 1-2. Excel データを1分でパワポ化する連携ワザ
- 出典URL: https://ai-advisors.jp/media/ai-news/claude-excel-powerpoint/
- 「このデータを集計して」「ピボットテーブル作って」と日本語で指示するだけで新しいシートが作られ、その分析サマリーが8枚程度のスライドに自動変換される、という資料作成フローの紹介。定例報告のたたき台づくりを大幅に短縮できます。数字の最終確認は人間が行う前提で使うのが安全です。

### 1-3. Claude Code で営業レポートを自動生成（CRM 連携）
- 出典URL: https://start-link.jp/hubspot-ai/ai/claude-code-practice/claude-code-sales-report-automation
- HubSpot（CRM＝顧客関係管理ツール）からデータを取得し、集計・分析・整形までを Claude Code で自動化。従来2〜3時間の作業を**5分以内**に短縮できたとする事例です。API 経由でデータソースに繋げば、レポート系はほぼ丸ごと委譲できることを示しています。

### 1-4. 22テンプレート×Claude Code でパワポ完全自動化
- 出典URL: https://note.com/keitaro_aigc/n/n20fc67dca970
- 企業名を入れるとコーポレートカラーを自動検出し、22パターンのスライドテンプレートから「見栄えの整った」資料を生成する仕組み。デザインの属人化（担当者しか作れない状態）を避け、体裁を標準化したいときの発想として参考になります。

> 適用シーン: インフラ運用の月次稼働報告・キャパシティレポート・障害サマリーなど、「データは手元にあるが清書が面倒」な作業。**入力（CSV/DB）→中間集計（Excel）→清書（PPT）**の3層で捉え、まず1本スキル化して定着させるのが定石です。

---

## 2. 非エンジニアも使える「Claude Cowork」の位置づけ

### 2-1. Claude Cowork とは（正式版・全有料プランへ）
- 出典URL: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork （公式）
- 補足解説: https://aismiley.co.jp/ai_news/claude-cowork-agent-os-vm/
- Cowork は Claude Code のエージェント能力を、デスクトップアプリの GUI で一般のナレッジワーカー向けに提供するもの。2026年1月にリサーチプレビュー、**4月10日に正式版**として全有料プランへ提供開始（Cowork 単体の追加料金なし、Pro でも利用可）。使い方は「Cowork タブでやりたいことを日本語で説明 → Claude の計画を確認 → 承認して実行」だけ。**ファイル削除・上書き・外部送信など不可逆な操作は計画を確認して承認する**設計で、これは NEXUS 運用の HITL（Human in the Loop＝人間が承認ループに入る仕組み）とも思想が一致します。

### 2-2. コネクタ連携でブラウザ操作まで
- 出典URL: https://cloudpack.jp/column/generative-ai/claude-cowork-guide.html
- Gmail・Google Drive・Slack・Notion・DocuSign など50以上のコネクタが提供され、Claude in Chrome と組み合わせればブラウザ操作を含む自動化も可能。**「重要ファイルは事前にバックアップ」**という基本対策も明記されており、着手前バックアップを標準運用にしている我々の方針とも合います。

---

## 3. ゼロデイ対策・セキュリティ運用の自動化（最新動向）

### 3-1.〔公式〕Claude Code Security が正式提供 ── 脆弱性の発見と修正提案
- 出典URL: https://www.anthropic.com/news/claude-code-security （公式）
- 2026年2月20日、Anthropic が Claude Code Security を公開。コードベースを走査して脆弱性を検出し、**人間レビュー前提で修正パッチを提案**する機能です。「発見だけでなく修正案まで出す（remediation loop＝修正の反復ループ）」点が従来ツールとの差で、Snyk も歓迎する評価を出しています（https://snyk.io/blog/claude-code-remediation-loop-evolution/ ）。

### 3-2. 「500件超のゼロデイ発見」とトリアージのボトルネック
- 出典URL: https://venturebeat.com/security/anthropic-claude-code-security-reasoning-vulnerability-hunting
- Frontier Red Team の研究で、Claude Opus 4.6 が本番 OSS から**500件超の高深刻度脆弱性**を発見・検証（長年の専門家レビューやファジングをすり抜けていたものを含む）。ただし論点は「発見が人間のトリアージ速度を追い越し、**発見から修正までの時間差こそが攻撃面になる**」こと。AI 推論による静的解析でノイズ（誤検知）を減らせる一方、**検証・修正の自動化を伴わせないと運用が詰まる**という示唆です（考察: https://futurumgroup.com/insights/claude-found-500-zero-days-who-patches-them-before-attackers-arrive/ ）。

> 注意点: 自動化する側も狙われます。前号でも触れた通り、出所不明の MCP／Skill を入れない・バージョンを上げ続ける、が最優先の防御です。

---

## 4. Playwright × Claude Code による E2E テスト自動化（日本企業の実績）

インフラ／Web 運用の「動作確認」を委譲する定番パターン。今週は国内テック企業の実績が充実していました。

### 4-1. GMOインターネットグループ ── Playwright MCP で E2E スキルを構築
- 出典URL: https://recruit.group.gmo/engineer/jisedai/blog/claude-code-playwright-e2e-test-automation/
- Claude Code と Playwright MCP（Model Context Protocol＝AI にツールを繋ぐ標準規格）を組み合わせ、UI からの動作確認の大半を自動化。テストを「スキル」として資産化する手法です。

### 4-2. ZOZO ── テストコードを書かない自然言語 E2E
- 出典URL: https://techblog.zozo.com/entry/claude-code-with-playwright-cli
- Claude Code + Playwright CLI で自然言語テストを実現。**膨大な組み合わせテストの自動化・計算検証の正確性・属人化解消**を同時に解決し、開発者が別案件を進めながら並行でテストを完了できる体制にした、という実務的な効果が報告されています。

### 4-3. KDDI ── UIリニューアルで約7,000項目、工数を約1/10へ
- 出典URL: https://tech-note.kddi.com/n/n4049604b3033
- 2026年1〜2月のデータチャージサイト UI リニューアルで合計約7,000項目を確認する必要があり、Claude Code と Playwright でテストコード作成からレポーティングまでを自動化。**試験工数を約1/10**にしたとされる大規模事例です。

### 4-4. kickflow ── ブラウザ操作を記録するだけのノーコード生成
- 出典URL: https://tech.kickflow.co.jp/entry/2026/02/24/101002
- Playwright Codegen（操作を記録してコード化する機能）と Claude Code を組み合わせた `/codegen-test` スキルで、画面をポチポチ操作するだけで整形済みテストが出力される仕組み。テスト作成のハードルを大きく下げます。

> 適用シーン: 監視ダッシュボードや管理画面のリグレッション確認（改修で既存機能が壊れていないかの検査）。**まず1画面を Codegen で記録 → Claude に整形・アサーション追加させる**と始めやすいです。

---

## さらに試す価値（awesome シリーズ・リンク集）

- ClaudeでExcel自動化｜アドイン＆Code 使い分け5活用法【2026】: https://uravation.com/media/claude-code-excel-automation-guide-2026/ ── Claude for Excel アドインと Claude Code の使い分けを整理。
- Claude for PowerPoint を試してみた（NEXTSCAPE blog）: https://blog.nextscape.net/archives/2026/03/17/094853 ── スライド自動生成〜既存資料の編集までの実操作レポート。
- Playwright MCP で E2E テストを自動化してみた（divx）: https://www.divx.co.jp/media/319 ── Playwright MCP 導入の入門記事。
- Claude Code と Playwright MCP で対話型 UI テスト（DevelopersIO）: https://dev.classmethod.jp/articles/building-interactive-ui-tests-with-claude-code-and-playwright-mcp/ ── 対話しながらテストを組む進め方。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
