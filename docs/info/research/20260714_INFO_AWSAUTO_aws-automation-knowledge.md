# AWS 業務自動化ナレッジ（週次・2026-07-14）

作成日: 2026-07-14 / STATUS: INFO / TOPIC: AWSAUTO

## 今週のテーマ / 見どころ

今週は「**お金・棚卸し系のレポート自動化**」と「**セキュリティ対応の自動修復（Auto-Remediation）**」を中心に、先週触れなかったサービスを広く拾いました。具体的には、コストの継続的な可視化（CUR→Athena→QuickSight）、AWS Config／GuardDuty による非準拠リソースの自動修復、そして 2025〜2026 年に大きく動いた IaC（Infrastructure as Code＝コードでインフラを管理する手法）事情（CDKTF 廃止、EventBridge Scheduler への cron 集約、生成 AI を使った運用調査の自動化）をまとめています。

用語のおさらい:
- **CUR**（Cost and Usage Report＝コストと使用状況レポート）: 全アカウント・全課金明細を S3 に毎日出力する“決定版”のコストデータ。
- **IaC**（Infrastructure as Code）: サーバやネットワークをコードで定義・再現する手法。
- **SRE**（Site Reliability Engineering＝サイト信頼性エンジニアリング）: 運用を工学的に自動化・改善する考え方。
- **Auto-Remediation**（自動修復）: 検知した設定違反や脅威を、人手を介さず自動で直す仕組み。

---

## 1. コスト / 棚卸しレポートの自動化（面倒なルーティンをなくす）

### 1-1. CUR → Athena → QuickSight でコストを“継続的に”自動可視化
出典: [Amazon QuickSight を使用した AWS CUR の可視化（前編）](https://aws.amazon.com/jp/blogs/news/amazon-quicksight-using-aws-cur-part1/) / [（後編）](https://aws.amazon.com/jp/blogs/news/amazon-quicksight-using-aws-cur-part2/) / [AWS Cost and Usage Reporting 公式](https://aws.amazon.com/aws-cost-management/aws-cost-and-usage-reporting/)

CUR が Parquet 形式で S3 に毎日届く → その上に Athena（サーバーレスのクエリエンジン）のテーブルを載せる → QuickSight（BI ダッシュボード）が Athena をデータソースにして描画、という構成です。ETL（データ変換処理）を書かずに「毎日更新されるコストダッシュボード」を作れるのが利点。AWS は Glue Crawler・Glue Database・Lambda を含む **CloudFormation テンプレート**を提供しており、Athena 連携の初期構築を自動化できます。月次のコスト報告書づくりを手作業からダッシュボード運用に置き換えたい時の定番パターンです。

### 1-2. 生成 AI にコストレポートを書かせる
出典: [生成AIを活用したAWSコストレポート（Qiita）](https://qiita.com/ts_pepeti/items/e53d54acd9becc06c255)

Cost Explorer / CUR から取得した数値を生成 AI に渡し、「増減の要因」「注目すべきサービス」を日本語の文章レポートに自動整形する事例。ダッシュボードの“数字”だけでなく、経営層や非エンジニア向けの“コメント付きサマリ”まで自動化できる点が実務的です。数値の解釈を人が毎月書く手間を削れます。

### 1-3. コスト可視化のリアルな運用知見
出典: [AWSのコスト可視化の取り組み（Zenn / Rentio）](https://zenn.dev/rentio/articles/aws-cost-visualization) / [Observability Best Practices: Cost Visualization](https://aws-observability.github.io/observability-best-practices/guides/cost/cost-visualization/cost/)

タグ設計（誰の・どの環境のコストか）を最初に決めておかないと、後から按分できず可視化が破綻する、という現場の教訓が語られています。棚卸し自動化の前提は「タグ運用の徹底」だと再認識できる内容です。

---

## 2. セキュリティ / 脆弱性・コンプライアンスの自動修復

### 2-1. Automated Security Response on AWS（ASR）でワンクリック修復
出典: [「AWS での自動化されたセキュリティ対応」の活用方法（builders.flash）](https://aws.amazon.com/jp/builders-flash/202409/automated-security-response-on-aws/) / [セキュリティレスポンス自動化の始め方（AWS ブログ）](https://aws.amazon.com/jp/blogs/news/how-get-started-security-response-automation-aws/)

ASR は AWS 公式のソリューションで、Security Hub の検出結果に対して**あらかじめ用意された修復（Playbook）**を、マルチアカウント・マルチリージョンで実行できます。「S3 バケットが公開されている」「暗号化が無効」といったよくある違反を、担当者が手で直す代わりにボタン操作／自動で是正できます。まず“定番の違反”を自動化の入口にするのがおすすめです。

### 2-2. AWS Config の非準拠リソースを自動修復
出典: [AWS Config を使用した非準拠リソースの修復（公式ドキュメント）](https://docs.aws.amazon.com/ja_jp/config/latest/developerguide/remediation.html) / [Config Rule 自動修復機能の紹介（DevelopersIO）](https://dev.classmethod.jp/articles/aws-config-auto-remediation/) / [Configでセキュリティ通知・自動修復を実装するポイント（NRIネットコム）](https://tech.nri-net.com/entry/security_notification_and_automatic_repair_function)

AWS Config は「あるべき設定」をルール化し、違反したリソースを **SSM Automation（Systems Manager の自動化ランブック）**で自動修復できます。実装は主に 2 系統: ①Lambda で直接直す、②Config のマネージド修復アクション＋SSM Automation を組み合わせる。NRI ネットコムの記事は「通知だけで止めるか、自動修復まで踏み込むか」の判断（誤修復リスクの見極め）に触れており、実運用で悩む所を押さえています。

### 2-3. GuardDuty × EventBridge × Lambda で脅威に自動対応
出典: [GuardDuty 編（CTC コラム）](https://www.ctc-g.co.jp/solutions/cloud/column/article/18.html) / [ConfigとGuardDutyは外せない（技術ブログ）](https://xkenshirou.hatenablog.com/entry/2020/12/12/100000)

GuardDuty（脅威検知）は CloudTrail・VPC フローログ・DNS ログを自動解析し、検出結果を EventBridge 経由で Lambda に流すことで、「不審な IAM アクセスキーの無効化」「該当インスタンスの隔離」などの一次対応を自動化できます。Config が“設定の正しさ”、GuardDuty が“振る舞いの怪しさ”を担当、と役割で覚えると設計しやすいです。

---

## 3. IaC / スケジューリング / 生成 AI 運用（2025〜2026 の最新動向）

### 3-1. 【要注意】CDKTF が 2025 年 12 月に廃止、CDK は機能拡張
出典: [AWS CDK vs Terraform: 2026 Comparison（Towards The Cloud）](https://towardsthecloud.com/blog/aws-cdk-vs-terraform) / [CDK vs Terraform 2026（DEV Community）](https://dev.to/aws-builders/aws-cdk-vs-terraform-the-complete-2026-comparison-3b4p)

HashiCorp が **CDKTF（TypeScript 等で Terraform を書く仕組み）を 2025 年 12 月に廃止**。一方で AWS CDK は **CDK Toolkit Library**（synth/deploy/rollback をコードから直接呼べる）、**CDK Refactor**、**CDK Mixins** を追加し、AWS ネイティブ用途では差を広げました。選定の目安は明快で、**AWS 単独なら CDK（言語・状態管理・抽象度が有利）、マルチクラウドなら Terraform/OpenTofu（宣言的 HCL・成熟したエコシステム・プロビジョニングが速い）**。既存で CDKTF を使っている場合は移行方針の見直しが必要です。

### 3-2. EventBridge Scheduler で「OS の cron」を卒業する
出典: [90本の cron バッチを一括移行（GAT技術ブログ）](https://www.g-a-t.co.jp/blog/entry/aws-batch-eventbridge-runcommand-ec2) / [EventBridge Scheduler でSSMオートメーションを定期実行（CloudBuilders）](https://www.cloudbuilders.jp/articles/2337/) / [公式ドキュメント](https://docs.aws.amazon.com/eventbridge/latest/userguide/using-eventbridge-scheduler.html)

各サーバの crontab に散らばった定期処理を、**EventBridge Scheduler（秒単位精度・柔軟なリトライ・月1000万リクエストまで無料枠）**へ集約する実践例。GAT の記事は 90 本の cron バッチを Scheduler＋Run Command＋EC2 に移行し、「どこで何が動いているか」を一元可視化しています。ノーコードで SSM Automation を定期実行し、EC2 の自動起動・停止やパッチ適用を回す使い方も定番です。タグベースで対象を指定すれば、インスタンスが増えても設定変更が不要になります。

### 3-3. 生成 AI で運用調査・障害対応を自動化（“権限を絞る”設計）
出典: [BTM 社事例: Bedrock × Strands Agents で調査自動化（AWS ブログ）](https://aws.amazon.com/jp/blogs/news/genai-case-study-btm/) / [CloudWatch 生成AIオブザーバビリティ GA（AWS）](https://aws.amazon.com/jp/about-aws/whats-new/2025/10/generative-ai-observability-amazon-cloudwatch/) / [AI に特権を与えず安全に Bedrock を活用した SRE 実践録](https://media.tcdigital.jp/ai-knowledge-flow/articles/aws-bedrock-sre-automation/)

運用の“調べる”作業を生成 AI に任せる動きが加速。BTM 社は Amazon Bedrock ＋コードエージェント（Strands Agents）で、Aurora・DynamoDB・CloudWatch を横断するシステム調査を自動化し、調査系の工数を大幅削減。設計の肝は **AI に読み取り権限のみを与え、「状況整理」と「対策提案」に役割を限定し、実際の変更は人間が承認する**こと（誤操作リスクの封じ込め）。2025 年 10 月には CloudWatch の生成 AI オブザーバビリティが一般提供（GA）となり、AI エージェント自体の監視もできるようになりました。NEXUS の HITL（人間が判断ループに入る）方針とも相性の良い設計思想です。

### 3-4. EC2 の起動・停止スケジュール自動化（コスト削減の即効ネタ）
出典: [EventBridge Scheduler で EC2 の自動起動・停止（Qiita）](https://qiita.com/Shoma0210/items/890965f1bc58c172c803)

開発・検証環境の EC2 を夜間・休日に自動停止するだけで、稼働費を大きく削れます。Scheduler ＋ Lambda（または SSM）で数十分で組める、最初の一歩に最適な自動化です。

---

## 4. さらに試す価値（awesome シリーズ / ハンズオン / リンク集）

- [awslabs/aws-well-architected-labs](https://github.com/awslabs/aws-well-architected-labs) — AWS 公式のハンズオン集。レベル 100（入門）〜400（上級）でコスト最適化・運用の自動化を手を動かして学べる。
- [mailtoharshit/awesome-aws-well-architected](https://github.com/mailtoharshit/awesome-aws-well-architected) — Well-Architected 関連リソースのキュレーションリスト。
- [aws-samples/well-architected-iac-analyzer](https://github.com/aws-samples/well-architected-iac-analyzer) — 生成 AI（Bedrock）で IaC や構成図を Well-Architected ベストプラクティスと突き合わせて評価するサンプル。設計レビューの自動化に。
- [aws-samples/sample-well-architected-skills-and-steering](https://github.com/aws-samples/sample-well-architected-skills-and-steering) — AI コーディングエージェントに Well-Architected を適用させるための再利用可能なスキル／ステアリング集（13 ツール対応）。
- [prowler-cloud/prowler](https://github.com/prowler-cloud/prowler) — マルチクラウドのセキュリティスキャナ。スキャン結果から Well-Architected Tool を自動更新する連携も議論されている。

---

## 参考: 今週の主な出典（まとめ）

- コスト自動化: [QuickSight×CUR 前編](https://aws.amazon.com/jp/blogs/news/amazon-quicksight-using-aws-cur-part1/) / [後編](https://aws.amazon.com/jp/blogs/news/amazon-quicksight-using-aws-cur-part2/) / [生成AIコストレポート](https://qiita.com/ts_pepeti/items/e53d54acd9becc06c255)
- セキュリティ自動修復: [ASR](https://aws.amazon.com/jp/builders-flash/202409/automated-security-response-on-aws/) / [Config 修復（公式）](https://docs.aws.amazon.com/ja_jp/config/latest/developerguide/remediation.html)
- IaC / 運用: [CDK vs Terraform 2026](https://towardsthecloud.com/blog/aws-cdk-vs-terraform) / [90本cron移行](https://www.g-a-t.co.jp/blog/entry/aws-batch-eventbridge-runcommand-ec2) / [Bedrock×SRE](https://media.tcdigital.jp/ai-knowledge-flow/articles/aws-bedrock-sre-automation/)

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
