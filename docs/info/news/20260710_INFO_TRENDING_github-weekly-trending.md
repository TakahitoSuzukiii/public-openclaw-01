# GitHub 週次トレンド（2026-07-10）

作成日: 2026-07-10 / STATUS: INFO / TOPIC: TRENDING

> データ元は GitHub 公式 REST API（Search API）です。Web スクレイピングは使用していません。前週比（delta）は本タスクが保持する前回スナップショット（2026-07-05）との差分から算出しています。掲載基準は原則スター 10,000 以上・開発者向け。優先トピック（go / rust / typescript / python / nextjs / claude / openclaw）は 1 万未満でも「注目枠」として扱います。
>
> 用語補足:
> - **スター（star）**: GitHub でリポジトリに付けられる「お気に入り」。人気の目安。
> - **前週比（delta）**: 直近 1 週間で増えたスター数。勢い（トレンド性）の指標。
> - **Skill（スキル）**: AI コーディングエージェント（Claude Code など）に手順書として読み込ませる再利用可能な指示セット。
> - **エージェント（agent）**: 目的を与えると自律的に複数手順を実行する AI プログラム。

---

## 今週の総括

今週の母集団は **356 リポジトリ**。トレンドの中心は先週に続き **「AI エージェント向け Skill（スキル）集」** です。前週比スター増（risers）の上位はほぼすべてが Claude Code などのエージェントに読ませる Skill／方法論リポジトリで占められました。個別プロダクトよりも「エージェントをどう賢く動かすか」というノウハウ集にスターが集まる流れが、より鮮明になっています。

優先トピックの動き:
- **claude**: 上位 risers を独占。addyosmani/agent-skills（+7,101）、mattpocock/skills（+7,081）が牽引。
- **python / typescript**: Skill 配布やスクレイピング API（firecrawl）で存在感。
- **rust**: ruvnet/RuView が Rust 枠のトップ（+3,192）。
- **openclaw**: graphify・hermes-agent の 2 件が該当（いずれもマルチエージェント対応の汎用ツール）。
- **go / nextjs**: 今週は 1 万超の新規急上昇は乏しく、nextjs は注目枠（後述）で紹介。

---

## 急上昇リポジトリ（risers）

### 1. addyosmani/agent-skills ⭐76,741（前週比 +7,101・JavaScript）
- **概要**: AI コーディングエージェント向けの「実務品質のエンジニアリング Skill」集。
- **なぜ注目か**: 今週の前週比トップ。Chrome チームでも著名な Addy Osmani 氏によるもので、信頼性の高い実装パターンをエージェントに渡せる点が支持されています。
- **主な特徴**: Claude Code / Codex / Cursor / Antigravity など複数エージェントに対応。
- **リンク**: <https://github.com/addyosmani/agent-skills>

### 2. mattpocock/skills ⭐164,481（前週比 +7,081・Shell）
- **概要**: TypeScript 界隈で著名な Matt Pocock 氏の `.claude` ディレクトリから公開された「本物のエンジニア向け Skill」集。
- **なぜ注目か**: 絶対スター数がトップクラスで、なお毎週 7,000 超の伸び。実務家の設定をそのまま覗ける点が人気。
- **主な特徴**: 実際の開発フローに沿った Skill を素の形で提供。
- **リンク**: <https://github.com/mattpocock/skills>

### 3. asgeirtj/system_prompts_leaks ⭐55,781（前週比 +5,957・JavaScript）
- **概要**: 各社 AI（Anthropic Claude、OpenAI、Google Gemini、xAI Grok など）のシステムプロンプトを抽出・収集したリポジトリ。
- **なぜ注目か**: プロンプト設計の「実物」を学べる資料として関心が高い。※あくまで第三者による抽出物であり、公式資料ではない点に留意。
- **主な特徴**: 主要モデルを横断的に収録、更新頻度が高い。
- **リンク**: <https://github.com/asgeirtj/system_prompts_leaks>

### 4. DietrichGebert/ponytail ⭐80,093（前週比 +5,357・JavaScript）
- **概要**: 「部屋で一番ものぐさなシニア開発者」のように考えさせる Skill。最良のコードは「書かずに済ませたコード」という発想。
- **なぜ注目か**: エージェントに過剰実装を抑えさせ、シンプルさを優先させる着眼点がウケています。
- **主な特徴**: 冗長な生成（over-engineering）の抑制に特化。
- **リンク**: <https://github.com/DietrichGebert/ponytail>

### 5. obra/superpowers ⭐251,713（前週比 +4,900・Shell）
- **概要**: 実際に機能するエージェント Skill フレームワーク兼ソフトウェア開発方法論。
- **なぜ注目か**: 絶対スター数が今回の掲載中で最大級。単発 Skill ではなく「方法論＋枠組み」として体系化されている点が評価。
- **主な特徴**: Skill を体系立てて運用するためのフレームワーク。
- **リンク**: <https://github.com/obra/superpowers>

### 6. VoltAgent/awesome-design-md ⭐100,567（前週比 +4,677）
- **概要**: 著名ブランドのデザインシステムを分析した `DESIGN.md` 集。プロジェクトに1つ置くだけで、コーディングエージェントに統一感のある UI を生成させられる。
- **なぜ注目か**: 「デザインの意図」を Markdown でエージェントに渡す新しいアプローチ。
- **主な特徴**: 各社デザインシステムの分析ドキュメントを収録。
- **リンク**: <https://github.com/VoltAgent/awesome-design-md>

### 7. firecrawl/firecrawl ⭐148,868（前週比 +3,844・TypeScript）
- **概要**: Web を大規模に検索・スクレイピング・操作するための API。
- **なぜ注目か**: 純粋なプロダクト系として上位に食い込んだ数少ない例。エージェントに「Web を読む目」を与える用途で定番化しつつある。
- **主な特徴**: 検索・取得・ページ操作を API 化。TypeScript 実装。
- **リンク**: <https://github.com/firecrawl/firecrawl>

---

## 新規ランクイン（newcomers）

今週プールに新しく入った 1 万超のリポジトリから抜粋します。

- **ChatGPTNextWeb/NextChat** ⭐88,437（TypeScript, nextjs/claude）— Web・iOS・macOS・Android・Linux・Windows 対応の軽量 AI アシスタント。マルチプラットフォームのクライアントとして根強い人気。<https://github.com/ChatGPTNextWeb/NextChat>
- **localsend/localsend** ⭐84,967（Dart）— AirDrop 代替のクロスプラットフォームなファイル共有。優先トピック外だが実用系の定番。<https://github.com/localsend/localsend>
- **Asabeneh/30-Days-Of-Python** ⭐67,861（Python）— Python を 30 日で学ぶ段階的ガイド。学習系の鉄板。<https://github.com/Asabeneh/30-Days-Of-Python>
- **sickn33/agentic-awesome-skills** ⭐42,805（Python, claude）— 1,900 以上の agentic Skill を集めたインストール可能なライブラリ。Skill ブームを象徴。<https://github.com/sickn33/agentic-awesome-skills>

---

## 注目枠（優先トピック）

スター 1 万未満・または個別トピックとして押さえておきたいものを別掲します。

### rust
- **ruvnet/RuView** ⭐79,866（前週比 +3,192・Rust, claude）— 市販 WiFi の電波から、映像を一切使わずに空間認識・バイタル計測・在室検知をリアルタイムに行う。Rust 枠の今週トップで発想が面白い。<https://github.com/ruvnet/RuView>

### openclaw
- **Graphify-Labs/graphify** ⭐81,891（前週比 +3,775・Python）— コード・SQL スキーマ・R/シェルスクリプト・文書・論文・画像などのフォルダを、Claude Code / Codex / OpenCode / Cursor / Gemini CLI 等で扱えるようにする AI コーディング支援 Skill。<https://github.com/Graphify-Labs/graphify>
- **NousResearch/hermes-agent** ⭐212,714（前週比 +3,088・Python）— 「使うほど育つ」を掲げる汎用エージェント。絶対スターが非常に大きい。<https://github.com/NousResearch/hermes-agent>

### nextjs
- **vercel/platforms** ⭐6,693（TypeScript）— マルチテナント対応のフルスタック Next.js アプリのテンプレート。Vercel 公式で、Next.js の実装リファレンスとして有用（1 万未満のため注目枠）。<https://github.com/vercel/platforms>
- **JCodesMore/ai-website-cloner-template** ⭐27,485（TypeScript, nextjs/claude）— コマンド1つで AI エージェントに任意サイトを複製させるテンプレート。<https://github.com/JCodesMore/ai-website-cloner-template>

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
