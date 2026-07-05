# GitHub 週次トレンド（2026-07-05）

作成日: 2026-07-05 / STATUS: INFO / TOPIC: TRENDING

> データ取得元: GitHub 公式 REST API（検索 API）。前回基準 2026-07-03 との差分（スター増・新規ランクイン）をもとに集計しています。掲載基準はスター 1万以上・開発者向け。優先トピック（go / rust / typescript / python / nextjs / claude / openclaw）は「注目枠」として別途取り上げます。
>
> 用語メモ: **スター（star）** = GitHub 上での「お気に入り登録」数で、人気の目安。**risers** = 前週比でスターが伸びたリポジトリ。**newcomers** = 今回新しく集計プールに入ったリポジトリ。**skill / スキル** = Claude Code などの AI コーディングエージェントに読み込ませて挙動を拡張する指示書ファイル群。

今週の傾向をひとことで言うと、**「AI エージェント用スキル（skill）」ブームが継続・過熱**しています。前週比の伸び上位はほぼ Claude Code 系スキルで占められました。トークン節約（caveman）、コード品質・美的センス（taste-skill / ponytail）、ペネトレーションテスト（strix）など、切り口が多様化しています。

---

## 🔎 注目枠（優先トピック）

優先トピックに該当するリポジトリを、言語・領域ごとに整理しました。

### claude（Claude Code 系スキル・エージェント）

今週もっとも動きが大きかった領域です。

- **JuliusBrussee/caveman** — ⭐ 84,779（前週比 **+1,946**） / 主要言語: JavaScript
  - 概要: 「なぜ多くのトークンを使う、少ないトークンで用は足りる」を地で行く、原始人風の短い言い回しで Claude Code のトークン消費を約 65% 削減するというネタ寄りのスキル。
  - なぜ注目か: LLM（大規模言語モデル）利用のコスト＝トークン量、という現実的課題をユーモアで解いており、実用とミームの両面でバズっています。
  - 特徴: プロンプトを極端に簡潔化 / 冗談めかしつつ効果は実測ベース。
  - リンク: https://github.com/JuliusBrussee/caveman

- **mattpocock/skills** — ⭐ 157,400（前週比 **+1,903**） / 主要言語: Shell
  - 概要: TypeScript 界隈で著名な Matt Pocock 氏が自身の `.claude` ディレクトリから公開している実務者向けスキル集。
  - なぜ注目か: 「実際に現場で使っている設定」という信頼性が支持を集めています。
  - 特徴: 実エンジニアの運用ノウハウをそのまま配布。
  - リンク: https://github.com/mattpocock/skills

- **Leonxlnx/taste-skill** — ⭐ 57,353（前週比 **+1,836**） / 主要言語: JavaScript
  - 概要: AI に「良いセンス（taste）」を与え、退屈で没個性な出力（いわゆる “slop”）を防ぐスキル。フロントエンド／デザイン寄り。
  - なぜ注目か: 生成物の“質感”という定量化しにくい部分に踏み込んでいる点が新しい。
  - リンク: https://github.com/Leonxlnx/taste-skill

- **DietrichGebert/ponytail** — ⭐ 74,736（前週比 **+1,822**） / 主要言語: JavaScript
  - 概要: AI エージェントを「部屋で一番怠惰なシニア開発者」のように振る舞わせる。YAGNI（You Aren't Gonna Need It＝必要になるまで作るな）思想に基づき、余計なコードを書かせない。
  - なぜ注目か: 「最良のコードは書かなかったコード」という設計哲学をエージェントに注入する発想。
  - リンク: https://github.com/DietrichGebert/ponytail

- **asgeirtj/system_prompts_leaks** — ⭐ 49,824（前週比 **+1,672**） / 主要言語: JavaScript
  - 概要: 各社 AI（Anthropic の Claude Fable 5・Opus 4.8、OpenAI、Google、xAI など）のシステムプロンプト（AI の土台となる内部指示文）を収集・公開しているリポジトリ。
  - なぜ注目か: プロンプトエンジニアリングの学習素材として需要が高い。※あくまで研究・教育用途としての参照に留めるのが無難です。
  - リンク: https://github.com/asgeirtj/system_prompts_leaks

- **affaan-m/ECC** — ⭐ 226,303 / 主要言語: JavaScript
  - 概要: エージェントの「ハーネス（実行基盤）」性能を最適化する仕組み。スキル・記憶・セキュリティ・リサーチ優先の開発フローを統合。
  - なぜ注目か: 個別スキルではなく、エージェント運用の“全体設計”に踏み込む大型プロジェクト。
  - リンク: https://github.com/affaan-m/ECC

- **anthropics/claude-code** — ⭐ 136,261 / 主要言語: Python
  - 概要: 本家 Anthropic による、ターミナルで動くエージェント型コーディングツール。
  - 特徴: コードベースを理解し、定型作業・コード解説・Git 操作を自然言語で実行。
  - リンク: https://github.com/anthropics/claude-code

- **anthropics/skills** — ⭐ 158,418 / 主要言語: Python
  - 概要: Anthropic 公式の Agent Skills 公開リポジトリ。上記のサードパーティスキル群の「本家」に当たる基盤。
  - リンク: https://github.com/anthropics/skills

### typescript

- **firecrawl/firecrawl** — ⭐ 145,024（前週比 **+1,351**） / 主要言語: TypeScript
  - 概要: Web を大規模に検索・スクレイピング・操作する API。AI エージェントに“Web を読む目”を与える用途で人気。
  - なぜ注目か: RAG（検索拡張生成）やエージェントの外部情報取得の定番基盤になりつつあります。
  - リンク: https://github.com/firecrawl/firecrawl

- **anomalyco/opencode** — ⭐ 182,609 / 主要言語: TypeScript
  - 概要: オープンソースのコーディングエージェント。Claude Code の対抗軸として言及されることが多い。
  - リンク: https://github.com/anomalyco/opencode

- **n8n-io/n8n** — ⭐ 195,301 / 主要言語: TypeScript
  - 概要: AI 機能を備えたワークフロー自動化プラットフォーム。400 以上の連携とセルフホストに対応。
  - リンク: https://github.com/n8n-io/n8n

### python

- **usestrix/strix** — ⭐ 36,997（前週比 **+2,550**／今週の伸び頭） / 主要言語: Python
  - 概要: アプリの脆弱性を発見・修正する、オープンソースの AI ペネトレーションテスト（侵入テスト）ツール。
  - なぜ注目か: 「AI × セキュリティ」領域での急伸。今週もっともスターを伸ばしました。
  - 特徴: バグバウンティ・レッドチーム・CTF ツールとしての利用を想定。
  - リンク: https://github.com/usestrix/strix

- **Panniantong/Agent-Reach** — ⭐ 51,224（前週比 **+1,426**） / 主要言語: Python
  - 概要: AI エージェントに「インターネット全体を見る目」を与える CLI。Twitter・Reddit・YouTube・GitHub・Bilibili・小紅書を API 料金ゼロで読める。
  - リンク: https://github.com/Panniantong/Agent-Reach

- **public-apis/public-apis** — ⭐ 446,866 / 主要言語: Python
  - 概要: 無料で使える公開 API の総覧リスト。定番中の定番。
  - リンク: https://github.com/public-apis/public-apis

### nextjs

- **langgenius/dify** — ⭐ 147,779 / 主要言語: TypeScript（topics に nextjs / python）
  - 概要: エージェント型ワークフロー開発の本番運用向けプラットフォーム。
  - なぜ注目か: ノーコード／ローコードで LLM アプリを組める統合基盤として継続的に支持。
  - リンク: https://github.com/langgenius/dify

### go

- **avelino/awesome-go** — ⭐ 177,268 / 主要言語: Go
  - 概要: Go 言語の優れたフレームワーク・ライブラリ・ソフトウェアのキュレーションリスト。Go 学習の入口として鉄板。
  - リンク: https://github.com/avelino/awesome-go

### rust / openclaw

- **openclaw/openclaw** — ⭐ 381,824 / 主要言語: TypeScript（topics に rust / openclaw）
  - 概要: 「あなた専用のパーソナル AI アシスタント。どの OS でも、どのプラットフォームでも」を掲げる自己ホスト型アシスタント基盤（＝本タスクの実行環境そのもの）。
  - なぜ注目か: openclaw トピックのエコシステムが拡大中。今週も継続して高い活動量。
  - リンク: https://github.com/openclaw/openclaw

> 補足: 今週、rust を主要言語とする 1万スター超の新規急伸リポジトリは、集計プール内では単独では目立ちませんでした（openclaw が rust トピックを含む形で該当）。

---

## 🚀 今週の伸び頭（risers トップ）

前週比でスターを大きく伸ばしたリポジトリ（優先枠と重複するものは簡略化）。

| リポジトリ | スター | 前週比 | 言語 | ひとこと |
|---|---|---|---|---|
| usestrix/strix | 36,997 | +2,550 | Python | AI 侵入テストツール |
| JuliusBrussee/caveman | 84,779 | +1,946 | JavaScript | トークン 65% 削減スキル |
| mattpocock/skills | 157,400 | +1,903 | Shell | 実務者スキル集 |
| Leonxlnx/taste-skill | 57,353 | +1,836 | JavaScript | AI に“センス”を付与 |
| DietrichGebert/ponytail | 74,736 | +1,822 | JavaScript | 怠惰なシニア思考 |
| asgeirtj/system_prompts_leaks | 49,824 | +1,672 | JavaScript | 各社システムプロンプト集 |
| Panniantong/Agent-Reach | 51,224 | +1,426 | Python | エージェントの Web 収集 CLI |
| firecrawl/firecrawl | 145,024 | +1,351 | TypeScript | Web スクレイピング API |
| obra/superpowers | 246,813 | +1,344 | Shell | エージェント開発方法論 |
| calesthio/OpenMontage | 33,605 | +1,245 | Python | エージェント動画制作システム |

補足で注目したいもの:

- **obra/superpowers**（⭐ 246,813 / +1,344）— サブエージェント駆動開発（subagent-driven development）を掲げるスキルフレームワーク兼開発方法論。
- **calesthio/OpenMontage**（⭐ 33,605 / +1,245）— 「世界初のオープンソース・エージェント型動画制作システム」を標榜。12 パイプライン・52 ツール・500 以上のスキルで、AI コーディングアシスタントを動画制作スタジオ化。

---

## 🆕 新規ランクイン（newcomers）

今回新しく集計プールに入った、注目のリポジトリ。

- **Graphify-Labs/graphify** — ⭐ 78,116 / Python（priority: python / claude / openclaw）
  - コード・SQL スキーマ・スクリプト・ドキュメント・画像・動画などを「クエリ可能なナレッジグラフ（知識グラフ）」に変換する AI コーディングアシスタント用スキル。Claude Code / Codex / OpenCode / Cursor / Gemini CLI などに対応。
  - リンク: https://github.com/Graphify-Labs/graphify

- **DeusData/codebase-memory-mcp** — ⭐ 26,668 / C（priority: claude）
  - 高性能なコード知能 MCP（Model Context Protocol）サーバ。コードベースを永続的なナレッジグラフに索引化し、平均的なリポジトリをミリ秒でインデックス。158 言語対応・サブミリ秒クエリ・トークン 99% 削減を謳う単一バイナリ。
  - リンク: https://github.com/DeusData/codebase-memory-mcp

- **practical-tutorials/project-based-learning** — ⭐ 272,181 / Python（priority: go / python）
  - 「作りながら学ぶ」プロジェクトベースのチュートリアル総覧。言語横断で実装課題を集約。
  - リンク: https://github.com/practical-tutorials/project-based-learning

- **massgravel/Microsoft-Activation-Scripts** — ⭐ 181,917 / Batchfile
  - Windows / Office のオープンソース アクティベータ。※用途の性質上、参照は自己責任・法令順守の範囲で。
  - リンク: https://github.com/massgravel/Microsoft-Activation-Scripts

- **fffaraz/awesome-cpp** — ⭐ 72,107 / （言語指定なし）
  - C++（および C）の優れたフレームワーク・ライブラリ・リソースのキュレーションリスト。
  - リンク: https://github.com/fffaraz/awesome-cpp

---

## まとめ

- 今週の主役は引き続き **AI コーディングエージェント用スキル**。伸び上位 10 のうち大半が Claude Code 系でした。
- 伸び頭は **usestrix/strix（+2,550）** で、「AI × セキュリティ（侵入テスト）」という実務直結の領域が伸長。
- 「トークン節約」「コード品質・美的センス」「怠惰＝過剰実装の回避」といった、**エージェントの“振る舞いを整える”系スキル**がトレンドの中心軸。
- ナレッジグラフ化（graphify / codebase-memory-mcp）による**コードベース理解の高速化・省トークン化**も新しい潮流として浮上。
- 優先トピックでは openclaw エコシステム（openclaw 本体・graphify・hermes-agent）の継続的な活性が確認できました。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
