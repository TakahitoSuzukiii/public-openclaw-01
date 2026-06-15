作成日: 2026-06-16 / STATUS: INFO / TOPIC: AWSAUTO

# AWS 業務自動化ナレッジ（週次・2026-06-16 号）

## 今週のテーマと見どころ

今週は「**定時バッチ運用の自動化**」と「**脆弱性管理の自動化**」を軸に集めました。EventBridge Scheduler（イベント駆動の定時実行サービス）＋ Lambda（サーバーレス関数）の組み合わせで、コスト通知・インスタンス自動停止・営業日判定までを“人手ゼロ”で回す事例が充実しています。あわせて、2026 年に大きく刷新された AWS Security Hub のマルチクラウド統合と、Amazon Inspector による自動脆弱性スキャンを取り上げます。末尾には実務で重宝する awesome 系リンク集も付けました。

用語の予習:
- **IaC**（Infrastructure as Code＝コードによるインフラ管理）: サーバやネットワークの構成をコードで宣言し、再現可能にする手法。
- **CSPM**（Cloud Security Posture Management＝クラウドのセキュリティ設定状態の継続監視）。
- **CVE**（Common Vulnerabilities and Exposures＝共通脆弱性識別子）: 既知の脆弱性に振られる世界共通の番号。
- **SES**（Simple Email Service＝AWS のメール送信サービス）。

---

## 1. ITインフラ運用の“面倒なルーティン”を自動化する

### AWS 利用料を Lambda + EventBridge Scheduler で自動監視・Slack 通知
出典: https://blog.serverworks.co.jp/notify-aws-cost-using-lambda-and-eventbridge

EventBridge Scheduler で定時トリガーを設定し、Lambda が Cost Explorer API（コスト・使用量を取得する API）から料金を読み出して Slack に自動投稿する構成です。「毎朝コンソールを開いて確認」という手作業を完全に置き換えられます。閾値超過時だけ通知する応用も容易で、コスト肥大の早期検知に有効。Lambda の実行は無料枠内に収まる規模が多く、運用コストはほぼ無視できます。

### EventBridge Scheduler × Systems Manager Automation で EC2/RDS を自動停止
出典: https://blog.serverworks.co.jp/how-to-stop-instance-by-tag

タグを目印に、夜間や休日に EC2/RDS を自動停止させる実装例。EventBridge Scheduler が定時に Systems Manager Automation（運用作業を手順書化して自動実行する機能）のランブックを呼び出します。検証・開発環境を“使う時間だけ起動”にするだけで、月額コストを数割削れるケースが多い、王道の節約パターンです。適用時はタグ設計を先に決めておくのがコツ。

### 祝日を判定して翌営業日に延期（Change Calendar × Lambda）
出典: https://dev.classmethod.jp/articles/changecalendar-holiday/

定期バッチを「祝日はスキップして翌営業日に回す」自動化。Systems Manager の Change Calendar（実行可否をカレンダーで管理する機能）と Lambda・EventBridge を組み合わせます。月末締め処理やレポート配信など“営業日基準”の業務をそのまま自動化でき、日本の祝日対応で悩みがちな運用に効きます。

### Cost Explorer API から Excel コストレポートを自動生成
出典: https://github.com/aws-samples/aws-cost-explorer-report

AWS 公式サンプル（aws-samples）の SAM 製 Lambda モジュール。Cost Explorer API のデータを基に、前月比グラフ付きの Excel レポートを自動生成します。経理・上長への定例報告を“ボタン操作なし”で配布でき、PPT/Excel 形式のレポート自動化を検討する際の出発点として最適。EventBridge + SES でメール配信まで繋げる派生事例（下記リンク集）も豊富です。

---

## 2. セキュリティ／脆弱性管理の自動化

### Amazon Inspector による自動脆弱性スキャンと優先度付け
出典: https://aws.amazon.com/inspector/features/

Amazon Inspector は EC2・コンテナイメージ・Lambda・コードリポジトリを継続的にスキャンし、ソフトウェア脆弱性とネットワーク露出を自動検出します。CVE 情報にネットワーク到達性や悪用可能性を掛け合わせ、文脈を考慮したリスクスコアを算出するため、「対応すべき脆弱性」の優先順位付けが自動化されます。検出結果は Security Hub と EventBridge に集約され、後段の自動化に流せます。

### Security Hub のオートメーションルールで検出を自動処理
出典: https://aws.amazon.com/about-aws/whats-new/2023/06/aws-security-hub-automation-rules

Security Hub の Automation Rules は、検出結果（findings）をほぼリアルタイムで自動更新・抑制できる機能。重大度の引き上げ、ワークフロー状態の更新、ノイズの自動抑制などをルールベースで実行でき、SOC（セキュリティ運用チーム）の手作業トリアージを大幅に削減します。

### Inspector × Systems Manager で脆弱性の検出から修復まで自動化
出典: https://aws.amazon.com/blogs/mt/automate-vulnerability-management-and-remediation-in-aws-using-amazon-inspector-and-aws-systems-manager-part-1/

AWS Cloud Operations Blog の実践ガイド。Inspector で検出した脆弱性を、Systems Manager のパッチ適用・ランブックで自動修復するパイプラインを解説します。「検出して終わり」ではなく“修復まで”を自動化できる点が実務的。ゼロデイ対応の初動短縮に直結します。

### 【2026 アップデート】Security Hub がマルチクラウド統合の SecOps 基盤へ
出典: https://aws.amazon.com/blogs/security/aws-security-hub-is-expanding-to-unify-security-operations-across-multicloud-environments/

2026 年、Security Hub は GuardDuty・Inspector・Macie・CSPM を単一体験に束ねる「統合セキュリティ運用」へと進化。脅威・脆弱性・設定ミス・機微データの各シグナルを横断的に継続解析し、マルチクラウド環境までカバーします。これまで複数コンソールに分散していた監視を一元化でき、運用負荷の削減効果が大きいアップデートです。

---

## 3. IaC／ツール選定（CDK・Terraform・CloudFormation）

### 2026 年版 IaC ツール比較ガイド
出典: https://go-cloud.io/terraform-vs-aws-cdk-vs-cloudformation/

CDK・Terraform・CloudFormation の使い分けを整理した 2026 年ガイド。要点は「唯一の正解はなく、スタックとチームのスキルで選ぶ」。CDK は CloudFormation を合成する“プログラミング言語ベース”で、L2 コンストラクト（ベストプラクティスを内包した部品）により定型コードを削減。Terraform はマルチクラウド対応かつ求人需要が高く、状態（state）管理を自前で制御できる柔軟性が強み。AWS 専業なら CDK/CloudFormation、複数クラウドや移植性重視なら Terraform、が大まかな指針です。

---

## 4. ベストプラクティス（Well-Architected）

### 運用上の優秀性（Operational Excellence）の柱
出典: https://wa.aws.amazon.com/wellarchitected/2020-07-02T19-33-23/wat.pillar.operationalExcellence.ja.html

Well-Architected フレームワーク 6 本柱のうち、自動化と最も関係が深いのが「運用上の優秀性」。**運用をコードとして実行**し、変更を小さく安全に行い、障害を見越して継続改善することを説きます。IaC 化・CI/CD・ランブック整備が核心で、本号で紹介した自動化事例はいずれもこの柱の実践例。自社構成の点検には、コンソールの Well-Architected Tool でセルフ評価するのが手軽です。

---

## さらに試す価値（awesome 系・リンク集）

- awesome-aws（定番の総合キュレーション）: https://github.com/donnemartin/awesome-aws
- awesome-aws-cost-optimization（2026 更新・コスト最適化特化）: https://github.com/gleesonb/awesome-aws-cost-optimization
- awesome-cloudops（CloudOps のツール／ベストプラクティス集）: https://github.com/Noovolari/awesome-cloudops
- awesome-aws-well-architected（Well-Architected 関連資料の集約）: https://github.com/mailtoharshit/awesome-aws-well-architected
- コストレポートをメール配信する派生実装（EventBridge + Lambda + SES + S3）: https://dev.to/bhatiagirish/automate-aws-cost-usage-report-using-event-bridge-lambda-ses-s3-aws-cost-explorer-api-3j3n
- Terraform + Lambda + Slack でコスト監視・閾値アラート: https://dev.to/copubah/automating-aws-cost-monitoring-with-terraform-lambda-and-slack-cem

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
