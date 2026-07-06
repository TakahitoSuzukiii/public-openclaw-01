作成日: 2026-07-06 / STATUS: INFO / TOPIC: AWSAUTO

# AWS 業務自動化ナレッジ（週次・2026-07-06）

## 今週のテーマ / 見どころ

今週は「**手順書（Runbook / ランブック）を人からコードへ渡す**」という軸で、AWS 公式ドキュメント・公式ブログを一次情報源に厳選しました。先週はコストレポート配信と Security Hub の自動仕分けを中心に扱ったので、今週はあえて別ジャンル——**運用手順の自動実行（SSM Automation）**、**脆弱性の自動修復（Inspector × SSM）**、**信頼性目標の自動監視（CloudWatch SLO）**、**イベント駆動のワークフロー（Step Functions × EventBridge）**——にフォーカスします。

見どころは次の3つです。

- **手順書をコード化して自動実行**: 「EC2 の再起動」「EBS のスナップショット取得」といった定型作業を、SSM Automation の Runbook（ランブック＝手順書）に落とし込み、承認・リトライ・スロットリング対策付きで回す。
- **脆弱性を見つけたら自動でパッチ**: Amazon Inspector が見つけた脆弱性（findings）を、SSM Automation で条件に応じて自動パッチ適用し、修復済みは自動で CLOSED にする。
- **信頼性を「目標」で測る自動化**: CloudWatch Application Signals の SLO（Service Level Objective＝サービス品質目標）で、過去データから目標値を自動レコメンドし、逸脱を GitHub ワークフローで検知する。

用語の基礎:
- **Runbook（ランブック）／Playbook（プレイブック）**: 障害対応や定型作業の「手順書」。AWS ではこれを機械可読の文書（SSM Document）にして自動実行できる。
- **IaC**（Infrastructure as Code＝コードによるインフラ管理）: 構成をコードで宣言し、再現・レビュー・テストできるようにする考え方。
- **SLO / SLI / エラーバジェット**: SLO は「稼働率99.9%」等の目標、SLI（Service Level Indicator）はそれを測る実測値、エラーバジェットは「許容できる失敗の残量」。
- **toil（トイル）**: 手作業で繰り返す・自動化可能・価値が増えない運用作業。SRE が減らそうとする対象。

---

## 1. ITインフラ運用のルーティン自動化（Operations as Code）

### 1-1. SSM Automation Runbook で運用タスクを解決する
出典: <https://aws.amazon.com/blogs/mt/use-aws-systems-manager-automation-runbooks-to-resolve-operational-tasks/>

AWS Systems Manager（SSM）Automation は、EC2 やその他 AWS リソースに対する定型作業を「Runbook（手順書）」として定義し、順次実行・条件分岐できる仕組みです。AWS が用意した数百のマネージド Runbook（再起動、AMI 作成、パッチ適用など）をそのまま呼ぶことも、独自 Runbook を書くこともできます。**「誰かが手順書を見ながら手で叩いていた作業」をそのまま自動化**できるため、対応のばらつき（人的ミス）を無くし、深夜・休日のオンコール負担を減らせます。運用の第一歩として最も費用対効果が高い自動化です。

### 1-2. EBS 関連の定型作業を Runbook で標準化
出典: <https://aws.amazon.com/blogs/mt/use-aws-systems-manager-automation-runbooks-to-resolve-elastic-block-store-related-operational-tasks/>

上記の実践編。EBS（Elastic Block Store＝EC2 のブロックストレージ）を題材に、SSM Document でボリューム構成を標準化し、バックアップ用スナップショットの取得を自動化し、性能・コスト要件に応じてボリューム属性を動的に変更する例が示されています。「棚卸し・バックアップ・構成変更」という**面倒だが定型的なストレージ運用**を、承認フロー付きで安全に回したい人向けです。

### 1-3. SSM Automation の実行制御が強化（2025年8月）
出典: <https://aws.amazon.com/about-aws/whats-new/2025/08/aws-systems-manager-automation-enhances-runbook/>

2025年8月のアップデートで運用が一段と楽になりました。①**コンソールから Runbook を再実行**（前回パラメータを引き継いで即リラン）、②高並列時に**スロットリングされた API 呼び出しを自動リトライ**して実行の信頼性を向上、③ターゲット指定で**ネストした OU（Organizational Unit＝組織単位）**を指定でき、複数アカウントをまたいだ細かなリソース制御が可能に。大規模フリートで Runbook を回す運用チームには効きます。なお**新規顧客は同日以降 Automation の無料枠が対象外**、既存顧客も2025年末で無料枠が終了する点は要注意（コスト計画に反映を）。

---

## 2. セキュリティ / 脆弱性管理の自動化

### 2-1. Inspector × SSM Automation で脆弱性を自動修復（Part 1）
出典: <https://aws.amazon.com/blogs/mt/automate-vulnerability-management-and-remediation-in-aws-using-amazon-inspector-and-aws-systems-manager-part-1/>

Amazon Inspector（EC2・コンテナ・Lambda を継続スキャンする脆弱性管理サービス）が見つけた findings を、SSM Automation の Runbook で**オンデマンドに一括修復**する構成です。入力パラメータで「修復したい深刻度（CRITICAL / HIGH …）」を選ぶ、または NONE を指定して当該インスタンスの全 findings を修復できます。パッチ適用に成功すると **Inspector が該当 findings を自動で CLOSED** にするため、「検知 → 修復 → 記録」までが人手を介さず閉じます。ゼロデイ・緊急パッチの初動を速くしたい運用に直結します。

### 2-2. タグベースで組織全体に横展開（Part 2）
出典: <https://aws.amazon.com/blogs/mt/automate-vulnerability-management-and-remediation-in-aws-using-amazon-inspector-and-aws-systems-manager-part-2/>

Part 2 は組織スケールの運用編。**リソースタグ**（resource tag＝リソースに付ける目印）と**深刻度フィルタ**で対象を絞り込み、AWS Organizations 配下の EC2 フリート全体を横断してパッチを当てます。「本番の CRITICAL だけ即修復」「特定チームのタグだけ対象」といった**ポリシーに沿った選択的自動修復**が組めます。SSM Patch Manager と組み合わせるのが定番構成です。注意点として、修復はビジネス影響（再起動等）を伴うため、メンテナンスウィンドウや承認ゲートと併用するのが安全です。

### 2-3. Security Hub カスタムアクションからワンクリック修復
出典: <https://aws.github.io/aws-security-services-best-practices/guides/inspector/>

Security Hub（各種セキュリティサービスの検出結果を集約する基盤）の**カスタムアクション**を使えば、集約された Inspector findings をコンソールで選び、ボタン一つで SSM Automation Runbook を起動して対象 EC2 群にパッチを当てられます。上記の公式ベストプラクティス集は、Inspector の有効化範囲・スキャン方式（エージェント型 / エージェントレス）・findings の運用まで網羅しており、**脆弱性管理を仕組みとして設計する**ときの土台になります。

---

## 3. 運用自動化（IaC / CI/CD / オブザーバビリティ / SRE）

### 3-1. CloudWatch Application Signals の SLO 自動化が大幅強化
出典: <https://aws.amazon.com/about-aws/whats-new/2026/03/cloudwatch-application-signals-adds-slo-capabilities/> / <https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-ServiceLevelObjectives.html>

Application Signals は、コード変更なしでアプリを自動計装（auto-instrumentation）し、サービスと依存関係を自動発見して SLI を集める APM（Application Performance Monitoring）機能です。2026年3月の更新で SLO 運用が一段と自動化されました。①**SLO レコメンデーション**（過去30日の P99 レイテンシとエラー率を分析し、妥当な目標値を提案）、②**サービスレベル SLO**（全オペレーションを横断した信頼性の俯瞰）、③**SLO パフォーマンスレポート**（日次・週次・月次でカレンダー整合の履歴分析）。「目標値を勘で決める」作業が要らなくなります。

### 3-2. SLO 違反を GitHub ワークフローで自動検知（AI 連携）
出典: <https://aws.amazon.com/about-aws/whats-new/2025/11/amazon-cloudwatch-application-signals-adds-github-action-mcp-server-improvements/>

2025年11月に GA した **GitHub Action** と MCP（Model Context Protocol＝AI にツールを繋ぐ規格）サーバ改善により、オブザーバビリティを開発フローに組み込めます。GitHub Actions のワークフロー内で**SLO 逸脱や重大エラーを検知**し、AI コーディングエージェントを使ってレイテンシ・エラー・SLO 違反の原因コードを特定できます。「リリース前に信頼性の劣化を自動で弾く」——シフトレフト（開発の早い段階で品質を担保する考え方）な運用に向きます。

### 3-3. Step Functions × EventBridge でイベント駆動オーケストレーション
出典: <https://aws.amazon.com/blogs/big-data/automate-and-orchestrate-amazon-emr-jobs-using-aws-step-functions-and-amazon-eventbridge/>

EventBridge（イベントルータ）で定時 or 条件をトリガに Step Functions（状態機械型のワークフロー）を起動し、多段処理を宣言的に回す定番パターンです。2025年の公式事例では、EventBridge の定時ルール → Step Functions が一時的な EMR クラスタを起動 → PySpark ジョブ実行 → 結果を S3 に書き戻し → **完了後クラスタを自動削除**、という「使い捨てバッチ基盤」を全自動化。**リトライ・分岐・待機を GUI/コードで管理**でき、Lambda だけでは辛い長時間・多段のジョブに向きます。使い捨て化でコストも抑えられます。

### 3-4. Well-Architected「安全に自動化する」の設計原則
出典: <https://docs.aws.amazon.com/wellarchitected/latest/operational-excellence-pillar/operational-excellence.html>

Well-Architected（AWS の設計指針集）の運用の優秀性（Operational Excellence）ピラーは、「**Safely automate where possible（可能な限り安全に自動化する）**」を柱に据えます。ワークロードと運用をコードで定義し、イベントに応じて自動起動する。その際は**ガードレール**——レート制御、エラー閾値、承認——を設けることで、暴走を防ぎつつ toil（トイル）と人的ミスを減らせます。「まず自動化ありき」ではなく「**安全装置付きの自動化**」という順序が重要、という設計思想の確認に。

---

## 4. さらに試す価値（awesome シリーズ / リンク集）

運用自動化のネタ探し・ツール選定に使えるキュレーション（厳選リンク集）です。全文転載はせず、出典のみ紹介します。

- **awesome-aws**（AWS ライブラリ・OSS・ガイドの総覧）: <https://github.com/donnemartin/awesome-aws> — CLI（aws-vault, awless 等）、監視（Netflix/ice）、耐障害性（SimianArmy / chaosmonkey）など運用ツールが分類済み。原本はやや古いため、コミュニティ・フォーク <https://github.com/sebastianmarines/awesome-aws> も併せて確認を。
- **awesome-cloudops**（CloudOps のツール・ベストプラクティス集）: <https://github.com/Noovolari/awesome-cloudops> — マルチクラウド前提で運用の観点別に整理されており、AWS 以外の視点も補える。
- **AWS Observability Best Practices**（オブザーバビリティ公式ベストプラクティス）: <https://aws-observability.github.io/observability-best-practices/tools/slos/> — SLO の立て方・計測・運用の実践ガイド。3-1/3-2 の設計に直結。
- **Serverless Workflows Collection**（Step Functions ワークフロー例の宝庫）: <https://serverlessland.com/workflows> — コピーして試せる公式サンプル集。3-3 の実装の出発点に。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
