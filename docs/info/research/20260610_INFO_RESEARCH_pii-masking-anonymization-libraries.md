# 機密情報マスキング（匿名化）処理 ライブラリ調査レポート

- **作成日:** 2026-06-10 (JST)
- **対象タスク:** タスクボード #20
- **目的:** セキュアに LLM を使うため、機密情報を含みうる生データを **LLM を介さず純コード（Python / TypeScript / Go / Rust）で匿名化** する関数を作る。そのための実績あるライブラリ・取り組みを調査（GitHub 中心）。
- **想定データ:** 個人情報・会社情報・クレジットカード情報などが混在する **SQL / JSON / TXT**。
- **プロトタイプ:** AWS Lambda 関数。
- **マスキング方針:** 本レポートに実データ・トークン・社内固有名は記載しない（placeholder のみ）。

---

## 0. 結論（先に要点）

- **本命は Microsoft Presidio（`microsoft/presidio`）**。PII（Personally Identifiable Information＝個人を特定できる情報）の検出・匿名化の事実上の業界標準。検出／編集／マスク／匿名化を text・画像・構造化データで提供し、要件適合度が最も高い。
- **軽量代替**として `scrubadub`（正規表現中心・軽量）、`DataFog`（高速・現代的 SDK）。
- **非 Python（TS/Go/Rust）** は成熟したエコシステムが Python より薄い。実績重視なら Python+Presidio が最有力。ランタイム制約で他言語必須の場合のみ別途精査。
- **AWS Lambda** は重い NLP モデル（spaCy/transformers）がサイズ制約に当たりやすい → **コンテナイメージ化**、または **正規表現＋パターン認識器中心の軽量構成**にする設計判断が必要。

---

## 1. 要件の整理

| 観点 | 要件 |
|---|---|
| 処理方式 | **LLM 非依存**の純コード（決定論的・再現可能・データ持ち出しなし） |
| 対象 | PII / 会社情報 / クレジットカード番号などが混在する **SQL・JSON・TXT** |
| 動作 | 特定の個人・会社を**特定できない文字列へ置換**（マスク／匿名化／仮名化） |
| 実行基盤 | プロトタイプは **AWS Lambda** |
| 言語 | Python 優先、無ければ TypeScript / Go / Rust |

> 用語: **マスキング**＝一部を伏字化（例 `4111-****-****-1111`）。**匿名化／仮名化**＝別の値へ置換し復元不可（または鍵管理下で復元可）。本件は両者を含む。

---

## 2. 本命: Microsoft Presidio

- **リポジトリ:** <https://github.com/microsoft/presidio>（★8,500+ / 活発に更新 / Python / MIT）
- **構成:**
  - `presidio-analyzer` … PII 検出。NLP（spaCy / transformers）＋正規表現＋チェックサム＋カスタム認識器（Recognizer）。
  - `presidio-anonymizer` … 検出箇所を replace / mask / hash / encrypt / redact 等で変換。
  - `presidio-structured` … **SQL/JSON 等の半構造化データを専用サポート**（列・キー単位の匿名化）。本要件に直結。
  - `presidio-image-redactor` … 画像中の PII（参考）。
- **標準で扱える主な実体:** 氏名・住所・電話・メール・**クレジットカード番号（Luhn チェック付）**・各種 ID（SSN 等）・日付・IP 等。日本固有項目（マイナンバー等）は**カスタム Recognizer**で追加。
- **AWS Lambda 雛形:** `colemvnio/starter-lambda-pii-anonymizer` 等のスターターが存在。
- **評価:** 検出網羅性・構造化対応・拡張性・実績すべてで最有力。

### 注意点（Lambda 化）
- spaCy / transformers モデルは重く、Lambda の **zip 250MB 制限**に収まりにくい。
- 対策: **(a) コンテナイメージで Lambda 化**（最大 10GB）／**(b) NLP を外し、正規表現・パターン認識器中心**にして軽量化。カード番号・メール・電話など「形が決まった PII」は NLP なしでも高精度に取れる。

---

## 3. 軽量な代替

| ライブラリ | リポジトリ | 特徴 | 向き |
|---|---|---|---|
| **scrubadub** | <https://github.com/LeapBeyond/scrubadub>（★400+ / 長期保守） | 正規表現＋任意で spaCy。TXT 向けで軽量・導入容易 | まず手早く TXT を匿名化 |
| **DataFog** | <https://github.com/DataFog/datafog-python>（活発・現代的） | 高速な正規表現を既定、任意で NLP/OCR/Spark。LLM ガードレール志向の SDK | 高速・大量・現代的 SDK が欲しい |

→ **Lambda の軽量プロトタイプ**には、scrubadub / DataFog の正規表現中心構成、または **Presidio を NLP 抜きで使う**のが現実的。

---

## 4. 非 Python（TypeScript / Go / Rust）

- 成熟・実績の面では **Python（Presidio）に明確な優位**。TS/Go/Rust にも PII 検出ライブラリは存在するが、構造化データ対応・カスタム認識器・実績の厚みは見劣りする。
- **判断:** 実績・SQL/JSON/TXT 対応・Lambda 実装の容易さを重視するなら **Python + Presidio が最有力**。Node ランタイム統一など**運用上どうしても他言語が必要な場合のみ**、候補を別途精査する。

---

## 5. 推奨アプローチ

1. **採用:** ベースに **Microsoft Presidio** を採用（`analyzer` + `anonymizer`、SQL/JSON は `presidio-structured`）。
2. **Lambda プロトタイプ:** まず **重 NLP を除外した「正規表現＋パターン認識器」構成**で軽量に実装 → 精度が要る項目（氏名・住所など文脈依存）が出たら **コンテナイメージ化**して NLP を追加。
3. **高精度化:** クレジットカード番号は **Luhn チェック付き Recognizer**、日本固有 PII（マイナンバー・郵便番号・電話）は**カスタム Recognizer**を追加。
4. **変換方針:** 復元不要なら `replace`/`hash`、復元要件があれば鍵管理下の `encrypt`（鍵は Secrets Manager / KMS、コード・ログに残さない）。
5. **入出力:** SQL はパース可能なら列指定、難しければ値抽出＋置換。JSON はキー指定の匿名化。TXT はエンティティ検出ベース。

---

## 6. 次アクション（要判断）

- 本レポートは調査まで。**PoC（Lambda + Presidio 軽量構成）の実装**は承認後に着手。
- 確定したら、比較表の精緻化・PoC 設計書・構築手順を `docs/openclaw/`（または本 research 配下）に記録する。
- 言語は Python 前提で進めてよいか（Node 統一等の制約有無）を確認したい。

---

## 7. 参考リンク（出典）

- Microsoft Presidio: <https://github.com/microsoft/presidio>
- Presidio ドキュメント: <https://microsoft.github.io/presidio/>
- scrubadub: <https://github.com/LeapBeyond/scrubadub>
- DataFog: <https://github.com/DataFog/datafog-python>
- Lambda PII スターター例: <https://github.com/colemvnio/starter-lambda-pii-anonymizer>

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
