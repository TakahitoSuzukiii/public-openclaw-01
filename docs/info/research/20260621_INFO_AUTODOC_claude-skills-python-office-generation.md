# 20260621_INFO_AUTODOC Claude Skills / Python / OpenClaw による資料自動生成（Excel・PowerPoint）

> タスクボード #80 の情報収集。定型業務・ルーティンワークの自動化に向け、**Excel(.xlsx) と PowerPoint(.pptx) の資料を自動生成する手段**を調査・比較し、1 手段を実地検証した記録。対象は Claude / Claude Agent Skills / Python / OpenClaw。作成日: 2026-06-21。

## 1. 背景・目的

週次レポートや定型資料の作成は手作業だと時間がかかる。Claude や Python、OpenClaw 連携の MCP を使って **Excel / PowerPoint を自動生成**できれば、データ抽出→整形→配布までを省力化できる。本書は選択肢を整理し、実際に 1 つの方式で週次レポートを自動生成して有効性を確認する。

## 2. 選択肢の整理（3 アプローチ）

### A. Claude Agent Skills（Anthropic 公式の文書スキル）

Anthropic は **文書作成・編集スキル**を公式提供している。Claude が動的に読み込み、専門タスクを再現性高く実行する「Skills（フォルダ化された手順＋スクリプト＋リソース）」の仕組み。

- 提供スキル: **pptx（PowerPoint）/ xlsx（Excel）/ docx（Word）/ pdf**。Claude の「ファイル作成機能」を支える実装。
- API では Beta ヘッダ（`anthropic-beta: skills-2025-10-02`）で Skills を有効化し、Excel/PowerPoint/Word/PDF を生成・編集できる。
- 公式リポジトリ `anthropics/skills` に docx/pdf/pptx/xlsx の実装が source-available で公開（Apache 2.0 のものと、文書スキルのように source-available のものが混在）。内部は **Open XML 処理 + LibreOffice headless** を併用。
- 公式: <https://platform.claude.com/docs/en/agents-and-tools/agent-skills/quickstart> / <https://github.com/anthropics/skills> / <https://claude.com/blog/skills>
- 長所: 高品質・自然言語指示で生成・Claude に統合。短所: API/Claude 環境前提、Beta、細かなレイアウト制御は内部実装依存。

### B. Python ライブラリ（フルコントロール）

- **openpyxl**: Excel(.xlsx) の生成・整形（数式・書式・条件付き書式・グラフ）。定型レポートの自動化に定番。
- **python-pptx**: PowerPoint(.pptx) をプログラムで生成。DB / 業務管理システムの内容から「プレゼン可能な状況報告」を自動作成する用途が代表例。テンプレートにデータを差し込むバッチ生成も一般的。
- pandas でデータ抽出 → openpyxl/python-pptx で出力 → cron / スケジューラで定期実行、という構成が王道。
- 公式: <https://python-pptx.readthedocs.io/> / <https://openpyxl.readthedocs.io/>
- 長所: 完全な制御・追加コストゼロ・オフライン。短所: レイアウトを自前で書く必要、pip 依存の導入が必要。

### C. OpenClaw 連携 MCP（本環境で利用可能）

本環境には資料生成系の MCP が連携済み（`/api/system-status` の「連携 MCP」でも確認可能）。

- **excel-mcp**: Excel ブック作成・データ書込・セル書式・テーブル/ピボット/グラフ作成等。
- **office-powerpoint-mcp**: PowerPoint の作成・スライド/表/グラフ追加・テンプレート適用等。
- 長所: **pip 導入不要（システム変更なし）**で即利用・Claude から直接呼べる・本環境に最適。短所: MCP サーバの機能範囲に依存。

### 比較まとめ

| 観点 | A. Claude Skills | B. Python | C. Office MCP |
|---|---|---|---|
| 導入コスト | API/Beta 前提 | pip 導入要 | **不要（連携済）** |
| 制御の細かさ | 中（内部依存） | **高** | 中〜高 |
| 自然言語生成 | **得意** | 自前 | 自前 |
| 本環境での即時性 | △ | △（要導入） | **◎** |

## 3. 実地検証（方式 C: excel-mcp で週次レポートを自動生成）

「システム変更を伴わない・ローカル新規追加のみ」のため、本環境で即利用できる **excel-mcp** を採用して検証した。

- **題材**: タスクボードの「トレーニング管理」データ（`/api/training/stats`）を集計し、**週次トレーニングレポート(.xlsx)** を新規自動生成。
- **生成物**: タイトル、KPI サマリ（トレ日数・総セット・総レップ・総ボリューム・直近7日）、部位別内訳、種目別トップ、曜日別ボリュームのシート。見出し・ヘッダに配色書式を適用。
- **パイプライン**: タスクボード API（集計）→ excel-mcp（ブック作成・データ書込・書式）→ ローカル保存（最小限）。
- **検証結果**: 生成ファイルは有効な OOXML（Microsoft Excel 2007+）。テキスト 56 セル・数値 36 セルが正しく格納され、全セクションを確認。**パイプラインは正常に機能**。
- **配置・プライバシー**: 生成レポートは**個人データ（身体・トレーニング）を含むためローカル限定**で保管し、公開リポジトリには出さない。本書（公開）には実数値を転記しない。

> 補足: 同様に office-powerpoint-mcp で .pptx の週次サマリも生成可能。Python（openpyxl / python-pptx）を使う場合は pip 導入（システム変更）となるため、事前承認のうえで実施する方針。

## 4. 推奨・運用方針

- **本環境では方式 C（Office MCP）を第一候補**。導入コストゼロで Excel/PowerPoint を即生成できる。
- 高度・大量・テンプレ差込が必要なら **方式 B（Python）** を pip 導入（要承認）で併用。
- Claude に自然言語で「この資料を作って」と任せる用途は **方式 A（Claude Skills）** が有力。
- 定期運用は cron で週次自動化（本タスクで設定。手順は別途構築ドキュメント参照）。
- **データの機微度に応じて出力先を分離**: 一般知識・手順 → 公開、身体/トレーニング等の個人データを含む生成物 → ローカル/private 限定。

## 5. 参考リンク

- Claude Agent Skills（Quickstart）: <https://platform.claude.com/docs/en/agents-and-tools/agent-skills/quickstart>
- anthropics/skills（公式スキル集・docx/pdf/pptx/xlsx）: <https://github.com/anthropics/skills>
- Introducing Agent Skills: <https://claude.com/blog/skills>
- python-pptx: <https://python-pptx.readthedocs.io/>
- openpyxl: <https://openpyxl.readthedocs.io/>

> 出典は調査時点の公開情報。仕様は変わりうるため、利用時は各公式ドキュメントを再確認すること。
