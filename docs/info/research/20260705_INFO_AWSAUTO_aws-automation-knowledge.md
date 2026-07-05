作成日: 2026-07-05 / STATUS: INFO / TOPIC: AWSAUTO

# AWS 業務自動化ナレッジ（週次・2026-07-05）

## 今週のテーマ / 見どころ

今週は「**運用のルーティンを『コード』と『イベント』に肩代わりさせる**」という視点で、AWS 公式ドキュメント・公式ブログを一次情報源に厳選しました。ポイントは次の3つです。

- **レポート配信の自動化**: コストレポートを人手でスクショ・整形・メール送信していた作業を、ダッシュボードのスケジュール配信や Lambda で丸ごと自動化する。
- **セキュリティの自動修復**: Security Hub の「オートメーションルール」で、検出結果（findings）の仕分け・重大度変更・チケット起票をリアルタイムに自動化する。
- **運用をコードにする（Operations as Code）**: パッチ適用・起動停止・手順書（Runbook / Playbook）をコード化し、人の手を介さず一貫して回す。

用語の基礎:
- **IaC**（Infrastructure as Code＝コードによるインフラ管理）: サーバやネットワークの構成をコードで宣言し、再現・レビュー・テストできるようにする考え方。
- **SRE**（Site Reliability Engineering＝サイト信頼性エンジニアリング）: 運用をソフトウェア的に自動化して信頼性を高める職能・方法論。
- **findings**（検出結果）: セキュリティサービスが見つけた問題の一件一件を指す。

---

## 1. ITインフラ運用のルーティン自動化

### 1-1. Instance Scheduler on AWS — EC2/RDS の起動停止でコスト削減
出典: <https://aws.amazon.com/solutions/implementations/instance-scheduler-on-aws/>

タグ（resource tag＝リソースに付ける目印のラベル）と Lambda を使い、EC2 と RDS（リレーショナルDB）を「業務時間外・週末は止める」といったスケジュールで自動起動停止する AWS 公式ソリューションです。使わない時間帯に止めるだけで、常時フル稼働と比べて費用が大きく下がります。クロスアカウント（複数 AWS アカウントをまたいだ）スケジューリング、自動タグ付け、CLI での節約額見積もりに対応。CloudFormation テンプレートで即デプロイでき、Well-Architected のコスト最適化ベストプラクティスに沿った定番の第一歩です。

### 1-2. EventBridge Scheduler の L2 Construct（CDK）で「止め忘れ」を根絶
出典: <https://aws.amazon.com/blogs/devops/announcing-the-general-availability-of-the-amazon-eventbridge-scheduler-l2-construct/>

上記のような起動停止を、自前の IaC で数行のコードにできる CDK（Cloud Development Kit＝コードでインフラを書くツール）の高レベル部品（L2 Construct）が GA（General Availability＝正式提供）済み。cron 式にタイムゾーン指定（例: `Europe/London` の 23:00）を直接書け、`ec2 stopInstances` などの API を「ユニバーサルターゲット」として呼べます。スケジュールをグループ単位で整理でき、Lambda への閲覧権限付与も1メソッド。**タイムゾーンを意識した定期実行**を宣言的に管理したい人向けです。

### 1-3. コストレポートのスケジュールメール配信（新機能）
出典: <https://aws.amazon.com/blogs/aws-cloud-financial-management/automate-aws-cost-reporting-with-scheduled-dashboard-email-delivery/> / <https://docs.aws.amazon.com/cost-management/latest/userguide/schedule-dashboard-reports.html>

Billing and Cost Management ダッシュボードの内容を、日次・週次・月次で PDF スナップショットにして関係者へ自動メール配信できるようになりました。コンソールへログインしてスクショを撮り、整形して役員へ送る——という FinOps（クラウド財務運用）チームの手作業がまるごと不要に。AWS コンソール権限のない CFO や部門長にも定期的にコストの可視性を届けられます。必要権限は `bcm-dashboards:CreateScheduledReport`。**PPT/Excel を手作業で作っていたレポート業務**の置き換え候補です。

### 1-4. マルチアカウントの「差分」コストレポートを Lambda で自動生成
出典: <https://aws.amazon.com/blogs/architecture/email-delta-cost-usage-report-in-a-multi-account-organization-using-aws-lambda/>

より自由度が欲しい場合の定番アーキテクチャ。EventBridge が定時に Lambda を起動 → Lambda が **Cost Explorer API** で各アカウントの費用を取得 → 前回との差分を計算・整形 → **Amazon SES**（メール送信サービス）で配信、という流れです。宛先は Lambda の環境変数で管理。組織（AWS Organizations）配下の複数アカウントの増減を一枚のメールにまとめられ、**独自フォーマット・独自集計**が必要なときに向きます。注意点として Cost Explorer API はリクエスト課金があるため、実行頻度は日次程度に抑えるのが無難です。

---

## 2. セキュリティ / 脆弱性管理の自動化

### 2-1. Security Hub オートメーションルールで findings を自動仕分け
出典: <https://docs.aws.amazon.com/securityhub/latest/userguide/automation-rules.html> / <https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-v2-automations.html>

Security Hub（複数のセキュリティサービスの検出結果を集約する基盤）は、取り込んだ findings を条件に応じて**リアルタイムに自動更新**できます。例:「本番アカウントの findings は重大度を HIGH→CRITICAL に引き上げる」「受容済みリスクの特定コントロールは自動で SUPPRESSED（抑制）にする」。従来は EventBridge ルール＋Lambda＋IAM 権限を自作・保守する必要がありましたが、**コンソール/API から直接ルールを組める**ようになり、セキュリティ・DevOps 担当の反復作業と MTTR（Mean Time To Response＝対応までの平均時間）を削減できます。管理者アカウントのみで作成でき、1アカウントあたり最大100ルール。

### 2-2. 自動化ルール設計のベストプラクティス
出典: <https://aws.amazon.com/blogs/security/streamline-security-response-at-scale-with-aws-security-hub-automation/>

公式ブログが挙げる要点: ①管理は管理者アカウントに集中（アクセス制御を厳格に）、②リージョン集約時はホームリージョンでルールを作ると全リージョンに適用、③マッチ条件は具体的に定義、④**ルール順序（RuleOrder）は数値が小さいほど先に適用**——順序の理解が重要、⑤内部の findings 更新はオートメーションルール、Lambda 呼び出しや SNS 通知など**外部アクションは EventBridge ルール**で（オートメーションルールが先に効く）、⑥抑制ルールは陳腐化しやすいので定期的に見直す。

### 2-3. Security Hub が GA、ニアリアルタイムのリスク分析へ
出典: <https://aws.amazon.com/about-aws/whats-new/2025/12/security-hub-near-real-time-risk-analytics/>

2025年12月、刷新版 Security Hub が GA。GuardDuty（脅威検知）・Inspector（脆弱性スキャン）・CSPM（Cloud Security Posture Management＝クラウド設定の姿勢管理）のシグナルを相関・強化し、**攻撃経路（attack path）の可視化**、セキュリティ視点のリソース棚卸し、チケット連携付きの自動レスポンスワークフローを提供します。複数コンソールを手で突き合わせる必要が減り、組織全体で**スケールする修復**が可能に。ゼロデイ・脆弱性対応の初動を速くしたい運用チームは要チェックです。

### 2-4. Systems Manager Patch Manager × メンテナンスウィンドウでパッチ運用を自動化
出典: <https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-patch-scheduletasks.html>

脆弱性対応の王道。パッチベースライン（適用してよい修正の定義）とパッチグループ（`Patch Group` タグで束ねた対象群）を用意し、**メンテナンスウィンドウ**（業務に影響しない時間帯の枠）に `AWS-RunPatchBaseline` ドキュメントを実行するタスクを登録すれば、Windows/Linux 横断で「スキャンのみ」または「スキャン＋適用」を定期実行できます。cron/rate 式で頻度を指定し、Concurrency（同時実行数）と Error Threshold（許容エラー数）で影響範囲を制御。CloudFormation の `AWS::SSM::MaintenanceWindowTask` で `AWS-PatchInstanceWithRollback` を使えば、ロールバック付きパッチ適用も IaC 化できます。注意: 適用後は原則再起動が入るため、`RebootOption` の扱いを事前に確認してください。

---

## 3. IaC / 運用をコードにするベストプラクティス

### 3-1. CDK ベストプラクティス — 構成は Construct、デプロイは Stack
出典: <https://docs.aws.amazon.com/cdk/v2/guide/best-practices.html> / <https://aws.amazon.com/blogs/devops/best-practices-for-scaling-aws-cdk-adoption-within-your-organization/>

公式が示す原則を押さえると保守性が段違いに上がります。①**Model with constructs, deploy with stacks**（論理単位は再利用可能な Construct にまとめ、Stack は「どう組み合わせて配置するか」だけを記述）、②設定は環境変数でなく**プロパティ/メソッドで渡す**（実行マシン依存を避ける）、③**インフラをユニットテスト**する（synth 時にネットワーク参照を避け、同じコミットが常に同じテンプレートを生む状態にする）、④**ステートフルなリソースの論理ID を変えない**（DB や S3 バケットは置換＝データ消失につながる）。組織展開では CDK Pipelines / GitHub Actions 用ワークフローで CI/CD を標準化し、開発者の認知負荷を下げるのが定石です。

### 3-2. Well-Architected「運用をコードとして実行する」× Runbook/Playbook
出典: <https://aws.amazon.com/blogs/mt/achieving-operational-excellence-using-automated-playbook-and-runbook/> / <https://docs.aws.amazon.com/wellarchitected/latest/operational-excellence-pillar/welcome.html>

Well-Architected の運用上の優秀性（Operational Excellence）ピラーの設計原則が「**Perform operations as code**（運用をコードとして実行する）」です。インフラだけでなく**運用手順そのものをコード化**（SSM Automation の Runbook＝定型作業の自動化手順、Playbook＝障害対応の手順書）することで、アプリコードと同様にテスト・検証でき、人の手による揺らぎを排して一貫性とスケーラビリティを得られます。実行ログが残るため、障害から学んで手順を継続改善する土台にもなります。**自己修復（self-healing）ワークロード**——問題を検知して人手なしで直す——を目指す出発点として最適です。

---

## さらに試す価値のあるリンク集

- Amazon Inspector（脆弱性管理サービス）の機能一覧 — EC2/ECR コンテナ/Lambda を継続スキャン: <https://aws.amazon.com/inspector/features/>
- Amazon Inspector ベストプラクティス（AWS 公式ガイド） — Security Hub のオートメーションルールとの連携: <https://aws.github.io/aws-security-services-best-practices/guides/inspector/>
- EventBridge Scheduler で Lambda を定期実行（公式チュートリアル）: <https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/with-eventbridge-scheduler.html>
- CDK Pipelines: AWS CDK アプリの継続的デリバリー: <https://docs.aws.amazon.com/cdk/v2/guide/cdk-pipeline.html>
- Instance Scheduler ソースコード（GitHub / aws-solutions）: <https://github.com/aws-solutions/instance-scheduler-on-aws/>

> 補足: 各記事の詳細な手順・コードは出典元をご参照ください。本記事は要点を日本語でまとめた紹介であり、全文転載ではありません。実装時は最新の公式ドキュメントで料金・リージョン対応・必要権限（IAM）を必ず確認してください。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
