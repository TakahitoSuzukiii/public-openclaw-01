# 週次まとめ：Claude / Claude Code による業務自動化ナレッジ（2026-07-13 号）

作成日: 2026-07-13 / STATUS: INFO / TOPIC: CLAUDEAUTO

> 注: 各記事の主張・数値・手順は出典元のものです。製品仕様やコマンドは変わり得るため、導入時は必ず一次情報で再確認してください。本記事は要約・考察であり全文転載ではありません。専門用語は綴りと意味を併記します。

## 今週のテーマ・見どころ

先週号（07-06）は「AI SRE・CI/CD・インフラ運用」のルーティン自動化を扱いました。今週は最優先トピックである **Excel／PowerPoint など「資料・レポート作成の自動化」** に軸足を戻し、実務での回し方を掘り下げます。目玉は、Claude が本物の `.xlsx` / `.pptx` / `.docx` ファイルを吐き出す **File Skills（ファイルスキル）** の仕組みと、テンプレート崩れを防ぐ「安全な差し込み設計」。あわせて、月次レポートや広告レポートの実測効果、企業ワークフローへの組み込み方、そして今週のセキュリティ枠として **Claude Code Security（脆弱性の自動発見・修正）** を紹介します。末尾には awesome シリーズのリンク集も。

用語メモ:
- **Skill（スキル）= モデルに「自社のやり方」を呼び出させる指示＋コード＋リソースの束**（汎用チャットボットを「自社の担当者」に近づける仕組み）
- **File Skills（ファイルスキル）= pptx / xlsx / docx / pdf など、実ファイルを生成・編集する組み込みスキル**
- **python-pptx = PowerPoint を Python から操作するライブラリ**
- **headless（ヘッドレス）= 画面（GUI）なしで自動実行するモード**
- **CI/CD = Continuous Integration / Continuous Delivery = 継続的統合／継続的デリバリー**
- **Files API = 生成ファイルを `file_id` 経由でダウンロードする API**
- **SAST = Static Application Security Testing = 静的アプリケーションセキュリティテスト**
- **PoC = Proof of Concept = 実証（ここでは攻撃可能性の実証コード）**

---

## 1. Excel / PowerPoint 資料作成の自動化（今週の主役）

### 1-1.〔公式〕Claude の File Skills 入門 ── 本物の Office ファイルを吐く仕組み
- 出典URL: https://platform.claude.com/cookbook/skills-notebooks-01-skills-introduction （公式 Cookbook）
- クイックスタート: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/quickstart （公式）
- Skill は「指示・実行コード・リソースをまとめた束」で、Claude に特定タスク専用の能力を持たせる仕組みです。組み込みの `pptx` / `xlsx` / `docx` / `pdf` を使うと、Claude が **本物の .xlsx / .pptx / .docx / .pdf を生成**します。API 経由では `skill_id`（例: `"xlsx"`, `"pptx"`）と `version`（`"latest"` 可）を指定し、生成物は応答に付く `file_id` を **Files API でダウンロード**する流れ。「なんとなくスライドっぽいもの」ではなく、後工程でそのまま配れるファイルが出る点が実務の肝です。

### 1-2. 組み込みスキルは本当に Excel/PPT を出せるのか？ 実測レビュー
- 出典URL: https://dev.to/miaoshuyo/can-claude-directly-output-real-excelppt-files-built-in-skills-tested-20l7
- 参考: https://support.claude.com/en/articles/12111783-create-and-edit-files-with-claude （公式ヘルプ）
- 「キーワード（PowerPoint / presentation 等）を検知して pptx スキルが起動し、タスクを論理的に構造化する」という挙動を実際に試した検証記事。Excel 側は **数式・書式・ゼロエラー検証**まで含めて生成でき、金融モデルのような「計算が合っていないと困る」用途にも耐えるかを見ています。導入前に「どこまで任せられるか」の肌感を掴むのに向いています。

### 1-3. Claude Code でパワポ自動化 ── 3手法と「デザイン崩れ」を防ぐ設計
- 出典URL: https://aurant-technologies.com/blog/claude-code-powerpoint-jidoka/
- 手法は3つ ──**公式スキル**（まず試す）／**python-pptx 連携**（会社テンプレ維持なら最有力）／**Microsoft 365 アドイン**（M365 導入済みなら）。実測で「月次3時間→30分以内」に短縮した例あり。崩れ防止の設計ルールが具体的で、**①既存テキストボックスへ差し込む ②グラフは"データのみ"更新 ③スライドマスターは編集しない** の3点を徹底。運用の要は「**全資料を一括自動化しない・1種類から始める**」「実行担当と"エラー時対応"まで含めて設計する」こと。いきなり全面展開せず段階導入、という現実解が参考になります。

### 1-4. Office ファイル生成を CI/CD に載せる ── claude-office-skills（GitHub）
- 出典URL: https://github.com/tfriedel/claude-office-skills
- Claude Code（CLI 版）向けに、PPTX（HTML→PPTX 変換・テンプレ編集）、Word（変更履歴＝tracked changes）、Excel（数式モデル）、PDF（フォーム入力・結合）をまとめた OSS スキル集。ポイントは **GUI なしの headless で、スクリプト／API／CI/CD パイプラインに組み込んで文書生成できる**こと。「月次レポートの定期生成」「大量ドキュメントのバッチ処理」を無人で回したい現場に向きます。資料作成を"人が押す作業"から"パイプラインの1ステップ"へ移せます。

---

## 2. 実運用の効果と「業務プロセスへの組み込み方」

### 2-1. 広告代理店の月次レポートを Claude Cowork で自動化してみた
- 出典URL: https://riddle-sense.co.jp/blog/claude-cowork-ad-report-automation/
- 「管理画面からデータをダウンロード → Excel に貼付・加工 → PowerPoint に転記」という、広告運用者が毎月手作業でやっていた定型フローを Cowork で丸ごと自動化した実録。**取得→加工→清書**という"転記地獄"こそ生成 AI の効き所、という好例です。自分の月次業務のどのステップが機械的な転記かを棚卸しする視点をくれます。

### 2-2. 企業レポート自動化の「型」── ワークフローの最終ノードにスキルを埋める
- 出典URL: https://agentskills.so/blogs/technical-practice-document-automation
- 参考: https://apidog.com/blog/claude-skills-for-document-processing/
- エンタープライズでの定石は、**業務プロセスの特定ノードにファイルスキルを埋め込む**こと。例：営業レポート自動化の"最終ステップ"で Excel スキルを明示的に呼び、構造化データを標準レポート形式で出力 → メールや社内システムで配布、という形で **AI 能力を業務フローに深く結線**します。「チャットで単発生成」から「業務パイプラインの1工程」へ格上げする設計思想が学べます。

### 2-3. 経理・営業での実測 ── 週20時間→週2時間の短縮例
- 出典URL: https://ai-advisors.jp/media/ai-news/claude-excel-powerpoint/
- 参考（M365 統合・Copilot 比較）: https://uravation.com/media/claude-excel-powerpoint-addin-microsoft365-2026/
- 2026年のアップデートで Excel・PowerPoint・Word の3アドインが**一つの会話コンテキストで連携**。経理部門で「部門別の集計→予算対比・前年同月比→エグゼクティブサマリーと実績グラフを自動生成」、営業で「Excel 分析→提案書ドラフトを Cowork で作成」した結果、**週20時間→週2時間**という短縮例が挙げられています。数値は出典元の事例値なので、自組織では小さく検証してから展開を。

---

## 3. 今週のセキュリティ枠：Claude Code Security（脆弱性の自動発見・修正）

### 3-1.〔公式〕Claude Code Security ── 防御側にフロンティア能力を
- 出典URL: https://www.anthropic.com/news/claude-code-security （公式）
- 参考（報道）: https://venturebeat.com/security/anthropic-claude-code-security-reasoning-vulnerability-hunting
- ルールベースのパターン照合ではなく、**人間のセキュリティ研究者のようにコードを"読んで推論"**するアプローチ。コンポーネントの相互作用を理解し、データフローを追跡し、複雑な脆弱性を検出します。すべての検出は **多段検証**を経て（Claude が各結果を再検査して誤検知＝false positive を除去）アナリストに届く設計。「大量アラートで疲弊する」既存 SAST の弱点を突く運用像です。

### 3-2. 500件超のゼロデイ発見と、その"次の一手"
- 出典URL: https://red.anthropic.com/2026/zero-days/ （公式 Frontier Red Team）
- 参考: https://futurumgroup.com/insights/claude-found-500-zero-days-who-patches-them-before-attackers-arrive/
- Frontier Red Team が本番 OSS で **500件超の高深刻度脆弱性**を発見・検証。ランダム入力を撒くファジングと違い、**コミット履歴を読んで"部分修正のバリアント（取りこぼし）"を探す**など、構造的に興味深い経路を狙う手口。防御側の含意はシンプルで、「見つける」より「**誰がいつ直すか（修正の運用）**」がボトルネックになる、という点。発見の自動化と同じ熱量で、パッチ適用フローの自動化を用意すべきという教訓です。

### 3-3. 修正ループへの統合 ── Snyk 視点のレビュー
- 出典URL: https://snyk.io/blog/claude-code-remediation-loop-evolution/
- セキュリティベンダー Snyk による、Claude Code を **remediation loop（検出→修正→再検証の輪）** に組み込む評価。エージェントに直すところまで任せる場合でも、**修正提案は人間レビュー前提**、テスト成功を完了条件にする、といったガードレールの重要性が語られます。§1 の資料自動化と同じ「人が最終判断を握る（human-in-the-loop）」思想が、セキュリティでも一貫している点に注目。

---

## さらに試す価値：awesome シリーズ / スキル集（リンク集）

- **VoltAgent/awesome-claude-code-subagents** ── 100+ の専門サブエージェント集: https://github.com/VoltAgent/awesome-claude-code-subagents
- **VoltAgent/awesome-agent-skills** ── 1000+ のエージェントスキル（Claude Code/Codex/Gemini CLI/Cursor 互換）: https://github.com/VoltAgent/awesome-agent-skills
- **hesreallyhim/awesome-claude-code** ── 定番の総合キュレーション（skills/agents/status lines/plugins）: https://github.com/hesreallyhim/awesome-claude-code
- **rohitg00/awesome-claude-code-toolkit** ── 135 agents / 35 skills / 176+ plugins 等の大型ツールキット: https://github.com/rohitg00/awesome-claude-code-toolkit
- **ComposioHQ/awesome-claude-skills** ── スキル特化のキュレーション: https://github.com/ComposioHQ/awesome-claude-skills
- **anthropics/claude-cookbooks（skills）** ── 公式のスキル実装例: https://github.com/anthropics/claude-cookbooks/blob/main/skills/README.md

> セキュリティ注意: awesome 系や外部スキルは**第三者コードを実行し得る**ため、導入前に中身を確認し、最小権限・サンドボックス・（可能なら）読み取り専用から。CI/CD へ組み込む場合は生成物と実行ログの監査（証跡）もセットで。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
