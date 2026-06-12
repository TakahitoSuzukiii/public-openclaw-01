# CloudFormation → AWS 構成図 自動生成ツール 選定レポート

- **作成日:** 2026-06-10 (JST)
- **対象タスク:** タスクボード #18
- **要件:** CloudFormation ソースから AWS 構成図を自動生成。**とにかく精度重視**・**無料**・**GitHub の OSS ベース**。

---

## 0. 結論（推奨）

| 順位 | ツール | 一言 |
|---|---|---|
| **★第一候補** | **awslabs/diagram-as-code（`awsdac`）** | **AWS 公式 Labs 製・公式アイコン準拠**。CFN テンプレを直接 PNG 構成図に変換（beta）。精度・公式準拠を最重視するならこれ |
| **第二候補（併用推奨）** | **mhlabs/cfn-diagram（`cfn-dia`）** | CFN/SAM/CDK → **編集可能な draw.io / インタラクティブ HTML / mermaid**。リソース種別フィルタが優秀。手直し前提・俯瞰用に最適 |

**おすすめ運用:** `awsdac` で公式準拠の図を生成 → 細部を直したい場合は `cfn-diagram` で draw.io 出力して編集。両方とも完全無料 OSS。

> ⚠️ CFN ソースからの自動生成は、どのツールでも「テンプレートに書かれた依存関係」しか描けない（暗黙の関係や論理グルーピングは推定）。**CFN 変換系は両者とも beta/発展途上**のため、最終図は軽微な手直しを前提にするのが現実的。

---

## 1. 候補比較

| ツール | 提供元 | 言語/導入 | 入力 | 出力 | ライセンス | 精度・特徴 |
|---|---|---|---|---|---|---|
| **awslabs/diagram-as-code** | **AWS 公式 Labs** | Go (`go install` / `brew install awsdac`) | YAML(DSL) ＋ **CFN テンプレ直接(beta)** | **PNG（公式AWSアイコン）** | Apache-2.0 | 公式アーキ図ガイドライン準拠。グループ自動配置。CFN→編集用YAML(dac)出力も可。MCPサーバ同梱 |
| **mhlabs/cfn-diagram** | OSS (mhlabs) | Node (`npm i -g @mhlabs/cfn-diagram`) | **CFN / SAM / CDK** | draw.io / vis.js HTML(対話) / **mermaid** / ascii | OSS(MIT系) | リソース種別・名前でフィルタ、JSON/YAML両対応、ネストスタック対応。**一部アイコン未対応** |
| Workload Discovery on AWS | AWS 公式ソリューション | CFNでデプロイ | **稼働中アカウントの実リソース** | Web上の対話図 | OSS | ソースではなく**デプロイ済み環境**を可視化。精度高いが要デプロイ・運用コスト大 |
| former2 | OSS | ブラウザ拡張/CLI | **稼働中アカウント** | 構成図＋IaC逆生成 | OSS | リバース用途。CFNソース入力ではない |
| terravision | OSS | Python | **Terraform のみ** | PNG（公式アイコン） | — | CloudFormation 非対応のため**対象外** |
| CloudForge | 商用SaaS | Web | 各種IaC | 各種 | **有料($19/mo)** | OSS でない・無料要件外のため**対象外** |

---

## 2. 第一候補: awslabs/diagram-as-code（awsdac）

- 出典: <https://github.com/awslabs/diagram-as-code>
- **AWS 公式 Labs 製** → アイコン・レイアウトが [AWS アーキテクチャ図ガイドライン](https://aws.amazon.com/architecture/icons) 準拠で**見た目の精度が高い**。
- CLI フラグ:
  - `-c, --cfn-template` … **CFN テンプレから直接ダイアグラム生成（beta）**
  - `-d, --dac-file` … CFN から **編集可能な dac(YAML) を生成**（→手直し後に再レンダリング可能）
  - `-o, --output` … 出力ファイル名（デフォルト `output.png`）
- 軽量・**ヘッドレスブラウザ/GUI 不要**で CI/CD に組み込みやすい。Go ライブラリとしても利用可。MCP サーバ同梱（AI 連携）。

### 導入・使用
```bash
# 導入（いずれか）
go install github.com/awslabs/diagram-as-code/cmd/awsdac@latest
# または
brew install awsdac

# CFN テンプレから直接 PNG 生成（beta）
awsdac template.yaml -c -o architecture.png

# CFN から編集用 dac(YAML) を生成 → 手直し → 再レンダリング
awsdac template.yaml -d -o diagram.yaml
awsdac diagram.yaml -o architecture.png
```

---

## 3. 第二候補: mhlabs/cfn-diagram（cfn-dia）

- 出典: <https://github.com/ljacobsson/cfn-diagram>（npm: `@mhlabs/cfn-diagram`）
- **CloudFormation / SAM / CDK** に対応。出力が豊富で**編集や CI ドキュメント化に強い**。
- 強み: リソース**種別・名前でのフィルタ**（IAM ロール/ポリシー等のノイズを除外して俯瞰しやすい）。JSON/YAML 両対応、ネストスタック対応。
- 出力形式:
  - `draw.io` … **編集可能**（VS Code Draw.io 拡張と相性良）
  - `html`（vis.js）… インタラクティブ
  - `mermaid` … README 埋め込み・CI ドキュメント向き（別パッケージ `cfn-diagram-ci` で画像化）
  - `ascii` … コンソールで即俯瞰
- 注意: **一部アイコン未対応**（公式準拠の見た目は awsdac が上）。

### 導入・使用
```bash
npm i -g @mhlabs/cfn-diagram

cfn-dia draw.io -t template.yaml      # draw.io（編集可能）
cfn-dia html -t template.yaml         # インタラクティブHTML
cfn-dia mermaid -t template.yaml      # mermaid
```

---

## 4. 選定の根拠（要件への適合）

- **精度重視:** 「公式アイコン・公式レイアウト準拠」という意味での精度は **awsdac（AWS 公式製）が最良**。「依存関係の網羅と手直し自由度」では **cfn-diagram の編集可能 draw.io** が補完。→ 両者併用が最も精度を高められる。
- **無料:** 両者とも完全 OSS・無料（awsdac=Apache-2.0、cfn-diagram=OSS）。クラウド課金なし。
- **GitHub OSS ベース:** 両者とも GitHub 公開リポジトリ。awsdac は AWS 公式組織 `awslabs` 配下で信頼性・メンテ性が高い。

## 5. 補足・注意

- CFN→図の自動化は「テンプレ記載の関係」に基づくため、論理的グルーピングや暗黙依存は完全再現できない場合がある。**awsdac/cfn-diagram の CFN 変換はともに beta/発展途上**。重要図は生成後の目視確認＋微修正を運用に含めること。
- 稼働中環境そのものを正確に可視化したい場合は **Workload Discovery on AWS**（要デプロイ・運用コスト）や **former2**（リバース）が別解。ただし本要件「CFN ソースから」とはアプローチが異なる。

## 6. 参考リンク（出典）
- awslabs/diagram-as-code: <https://github.com/awslabs/diagram-as-code>
- CFN 変換ドキュメント: <https://github.com/awslabs/diagram-as-code/blob/main/doc/cloudformation.md>
- mhlabs/cfn-diagram: <https://github.com/ljacobsson/cfn-diagram>
- Workload Discovery on AWS: <https://docs.aws.amazon.com/solutions/latest/workload-discovery-on-aws/solution-overview.html>
- AWS アーキテクチャアイコン: <https://aws.amazon.com/architecture/icons>
