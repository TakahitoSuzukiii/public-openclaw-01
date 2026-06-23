作成日: 2026-06-24 / STATUS: INFO / TOPIC: AUTODOC

# 資料自動生成ナレッジ（Excel / PowerPoint）— 週次まとめ

今週のテーマ: **「Python ライブラリの設計思想」と「MCP サーバの実装地図」**

先週・先々週は Claude Code を使った PowerPoint / Excel 自動生成に焦点を当てました。今週は切り口を変え、自動化の土台となる **Python ライブラリ（openpyxl / python-pptx / pandas）をどう使い分けるか**という設計の観点と、AI エージェントから Office を操作する **MCP（Model Context Protocol、AI とツールをつなぐ標準プロトコル）サーバの全体像**を整理します。「どのツールを、どの役割で組み合わせるか」を理解すると、自前の定期レポート自動化がぐっと作りやすくなります。

---

## アプローチ A：Python ライブラリ（Excel 側）— openpyxl / pandas / XlsxWriter の使い分け

### A-1. openpyxl と pandas は「役割分担」で組み合わせる
- 出典: [How to use Python and Excel together with pandas and openpyxl (Stringfest Analytics)](https://stringfestanalytics.com/how-to-use-python-and-excel-together-with-pandas-and-openpyxl/) / [Real Python: A Guide to Excel Spreadsheets with openpyxl](https://realpython.com/openpyxl-excel-spreadsheets-python/)
- pandas は「データの集計・加工」、openpyxl は「セル単位の書式・グラフ・ブック構造の細かい制御」が得意です。実は **pandas は内部で openpyxl をバックエンドとして使う**ため、両者は競合ではなく分業の関係です。
- 定石ワークフロー：複数シートを pandas の DataFrame（表形式データ構造）に読み込み → pandas で集計 → openpyxl で条件付き書式やグラフを付けた最終レポートに仕上げる。
- 適用シーン: 週次の売上・稼働レポート。注意点として、書式を凝るなら DataFrame 目線（pandas）ではなく「Excel 文書」目線（openpyxl）に頭を切り替える必要があります。

### A-2. 書き込みライブラリの三択：pandas / XlsxWriter / openpyxl
- 出典: [Excel File Writing Showdown: Pandas, XlsxWriter, and Openpyxl (Medium)](https://medium.com/@badr.t/excel-file-writing-showdown-pandas-xlsxwriter-and-openpyxl-29ff5bcb4fcd) / [OpenPyXL vs Pandas vs Spire.XLS (Medium, 2026-01)](https://medium.com/@alexaae9/openpyxl-vs-pandas-vs-spire-xls-which-one-should-you-use-for-excel-automation-8f2c42c1cc8e)
- ざっくり指針：**単純な書き出しなら pandas の `to_excel()`、新規ファイルへの高速書き込み＋グラフは XlsxWriter、既存ファイルの読み書き・編集は openpyxl**。
- 注意点: XlsxWriter は「書き込み専用」で既存ファイルを読めません。逆に既存テンプレートを開いて追記するなら openpyxl 一択です。用途を最初に見極めると手戻りが減ります。

### A-3. 入門〜実務の基礎固め
- 出典: [Excel Automation with Openpyxl (GeeksforGeeks)](https://www.geeksforgeeks.org/python/excel-automation-with-openpyxl-in-python/) / [openpyxl: Automate Excel Tasks with Python (DataCamp)](https://www.datacamp.com/tutorial/openpyxl)
- セルの読み書き、数式の埋め込み、条件付き書式、グラフ生成までを体系的に学べます。財務レポートの自動化、DB エクスポートからのグラフ生成といった定番ユースケースが具体例付きで紹介されています。
- 適用シーン: 「まず動くものを作る」段階の参照に最適。手を動かしながら API の勘所をつかめます。

---

## アプローチ B：Python ライブラリ（PowerPoint 側）— python-pptx のテンプレ駆動

### B-1. テンプレ（.pptx）+ プレースホルダ ID で「会社書式を壊さない」自動化
- 出典: [python-pptx を使って組織テンプレートに従ったスライド自動作成 (Qiita)](https://qiita.com/a-yamagata/items/7c87b2cfd6c515e2aa65)
- 既存の会社テンプレートを「型」として読み込み、各レイアウトの **プレースホルダ ID（差し込み枠の番号）** を洗い出して、そこにデータを流し込む手法です。
- 強み: フォント・ロゴ位置・背景装飾が会社規約に完全準拠。デザイン変更は**テンプレ .pptx を直すだけ**で済み、Python コードを触らなくてよいのが運用上の最大の利点です。
- 適用シーン: 毎週フォーマットが固定の定例レポート。注意点として、最初にプレースホルダ ID を特定する「下調べスクリプト」を一度書いておくと以後が楽になります。

### B-2. テンプレ機能を足す `python-pptx-templating`
- 出典: [python-pptx-templating (PyPI)](https://pypi.org/project/python-pptx-templating/)
- python-pptx に **mustache（`{{name}}` 形式の差し込み記法）テンプレート**機能を追加するライブラリ。マスタースライドの枠に名前を付け、辞書データを流し込むだけでスライドを量産できます。
- 適用シーン: プレースホルダ ID を意識せず「名前で差し込みたい」場合に便利。注意点として、複雑なレイアウトには素の python-pptx の方が小回りが利きます。

### B-3. 10行スケールから始める入門
- 出典: [Python×PPTXで時短：たった10行で資料を自動生成 (chocottopro)](https://chocottopro.com/?p=300) / [Pythonを使ったレポートの自動作成【PowerPoint】(Qiita)](https://qiita.com/kousakulog/items/34855cd8286bd4f33c08)
- スライド追加・レイアウト指定・タイトル/本文の流し込みという最小構成から、週次の精度レポートのような定例物まで段階的に学べます。
- 適用シーン: 「python-pptx を初めて触る」段階の最短ルート。

---

## アプローチ C：pandas → グラフ → PowerPoint の橋渡し

### C-1. matplotlib 画像を貼る方式 と ネイティブグラフ方式
- 出典: [Creating PowerPoint Presentations with Python (Practical Business Python)](https://pbpython.com/creating-powerpoint.html) / [Automate PowerPoint Slides Creation with Python (Towards Data Science)](https://towardsdatascience.com/automate-powerpoint-slides-creation-with-python-a639c7d429a6/)
- グラフの入れ方は2系統：**(1)** matplotlib で描画 → 画像保存 → スライドに貼り付け、**(2)** python-pptx の `CategoryChartData` + `XL_CHART_TYPE` でネイティブなグラフを直接生成。
- 使い分け: 凝った見た目なら (1)、PowerPoint 上で後から数値編集したいなら (2)。データ抽出→可視化→スライド化までを 1 本のスクリプトで完結できます。

### C-2. DataFrame を表スライドに変換
- 出典: [PandasToPowerpoint (GitHub)](https://github.com/robintw/PandasToPowerpoint) / [7 Ways to Boost Productivity with Python PowerPoint Automation (SoftKraft)](https://www.softkraft.co/python-powerpoint-automation/)
- python-pptx は DataFrame を直接「表」にする標準 API を持たないため、`PandasToPowerpoint` のようなユーティリティが DataFrame → PowerPoint テーブル変換を補ってくれます。
- 適用シーン: 集計結果をそのまま表スライドにしたいレポート。

---

## アプローチ D：MCP サーバの実装地図（AI エージェントから Office を操作）

自前コードを書かず、AI エージェント（Claude Code / Claude Desktop 等）から自然言語で Office を操作する選択肢です。実装方式で性格が大きく分かれます。

- **COM 自動化系（Windows 必須・実アプリ操作）**
  - [sbroenne/mcp-server-excel](https://github.com/sbroenne/mcp-server-excel)（23 ツール / 214 操作、Windows COM で実 Excel を制御）
  - [jenstangen1/pptx-xlsx-mcp](https://github.com/jenstangen1/pptx-xlsx-mcp)（pywin32 で起動中の Office に介入）
  - 特徴: 実アプリを動かすので再現性が高い反面、Windows + Office インストールが前提。

- **ファイル直接生成系（OS 非依存・Office 不要）**
  - [dvejsada/mcp-ms-office-documents](https://github.com/dvejsada/mcp-ms-office-documents)（Docker で pptx/docx/xlsx/eml を生成）
  - [iOfficeAI/OfficeCLI](https://github.com/iOfficeAI/OfficeCLI)（単一バイナリ・Office 不要、AI ツールの設定を自動検出して skill を導入）
  - 特徴: サーバ常駐や CI に組み込みやすい。OpenClaw 連携の `excel-mcp` / `office-powerpoint-mcp` も本系統で、**pip 追加なしで動く**のが運用上の利点。

- **スイート/Graph 連携系**
  - [lingfan36/ai-office-mcp](https://github.com/lingfan36/ai-office-mcp)（PPT/Excel/Word を束ねた最も網羅的な OSS、Claude Code/Cursor/Cline 対応）
  - [Aanerud/MCP-Microsoft-Office](https://github.com/Aanerud/MCP-Microsoft-Office)（Microsoft Graph 経由で M365 全体に 117 ツール）
  - 特徴: Office 単体を超え、メール・カレンダー・Teams まで含めた業務全体の自動化に向く。

選び方の指針: **OS 非依存・サーバ常駐なら「ファイル直接生成系」、既存の複雑なマクロ資産を活かすなら「COM 系」、M365 全体を巻き込むなら「Graph 系」**。本環境（OpenClaw + サーバ常駐）ではファイル直接生成系が素直です。

---

## 参考リンク集
- openpyxl × pandas 役割分担: <https://stringfestanalytics.com/how-to-use-python-and-excel-together-with-pandas-and-openpyxl/>
- Real Python openpyxl ガイド: <https://realpython.com/openpyxl-excel-spreadsheets-python/>
- 書き込みライブラリ比較: <https://medium.com/@badr.t/excel-file-writing-showdown-pandas-xlsxwriter-and-openpyxl-29ff5bcb4fcd>
- openpyxl vs pandas vs Spire (2026-01): <https://medium.com/@alexaae9/openpyxl-vs-pandas-vs-spire-xls-which-one-should-you-use-for-excel-automation-8f2c42c1cc8e>
- openpyxl 入門 (GeeksforGeeks): <https://www.geeksforgeeks.org/python/excel-automation-with-openpyxl-in-python/>
- python-pptx テンプレ駆動 (Qiita): <https://qiita.com/a-yamagata/items/7c87b2cfd6c515e2aa65>
- python-pptx-templating: <https://pypi.org/project/python-pptx-templating/>
- python-pptx 10行入門: <https://chocottopro.com/?p=300>
- pandas → PowerPoint (Practical Business Python): <https://pbpython.com/creating-powerpoint.html>
- Automate PowerPoint (Towards Data Science): <https://towardsdatascience.com/automate-powerpoint-slides-creation-with-python-a639c7d429a6/>
- PandasToPowerpoint: <https://github.com/robintw/PandasToPowerpoint>
- Office MCP まとめ (Awesome MCP Servers): <https://mcpservers.org/servers/jenstangen1/pptx-xlsx-mcp>
- ai-office-mcp: <https://github.com/lingfan36/ai-office-mcp>
- OfficeCLI: <https://github.com/iOfficeAI/OfficeCLI>
- mcp-ms-office-documents: <https://github.com/dvejsada/mcp-ms-office-documents>

---

注: 本記事は一般公開情報の要約・考察であり、全文転載は行っていません。出典各 URL を参照してください。
