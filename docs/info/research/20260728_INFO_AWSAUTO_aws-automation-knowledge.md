# AWS 業務自動化ナレッジ（週次）— ワークフロー(Step Functions)・設定違反の自動修復・脆弱性管理AI化

作成日: 2026-07-28 / STATUS: INFO / TOPIC: AWSAUTO

## 今週のテーマ / 見どころ

先週（07-21）は「生成AIに運用を任せる（Amazon Q Developer）」「鍵レス CI/CD（OIDC）」「SSM 実務」を扱いました。今週は重複を避け、**"つなぐ・直す・守る" を自動化する** 3本柱で集めています。

1. **つなぐ（オーケストレーション）** — AWS Step Functions で複数サービスをワークフロー化。2026年6月には AI エージェントの推論ステップまで組み込めるようになりました。
2. **直す（自動修復）** — AWS Config で「設定違反リソース」を検知したら、人手を介さず自動で直す（Auto-Remediation）。棚卸し・監査の手戻りを消します。
3. **守る（脆弱性管理の AI 化）** — Amazon Inspector / GuardDuty。GuardDuty には調査を数分に短縮する AI 調査エージェントが登場。**Inspector Classic は 2026-05-20 に廃止済み**という重要な移行ポイントも押さえます。

用語の下ごしらえ:
- **IaC** = Infrastructure as Code = インフラ構成をコードで管理する考え方（CloudFormation/CDK/Terraform）。
- **オーケストレーション** = 複数の処理・サービスを順序や条件付きで束ねて動かすこと。
- **Auto-Remediation**（自動修復）= 検知した設定違反や脅威を、人手を介さず自動で直す仕組み。
- **ステートマシン** = Step Functions における「状態（処理ステップ）の集合＝ワークフロー本体」のこと。
- **CVE** = Common Vulnerabilities and Exposures = 既知の脆弱性に付く共通の識別番号。

---

## 1. つなぐ — AWS Step Functions でワークフローを自動化する

### 1-1. Step Functions に「AIエージェントの推論ステップ」が追加（2026-06）
出典: <https://aws.amazon.com/jp/about-aws/whats-new/2026/06/aws-step-functions-agentcore/>

Step Functions は、Lambda・ECS・SNS など複数の AWS サービスを「状態（ステップ）」としてつなぎ、順序・分岐・リトライ（自動再試行）・エラー処理まで宣言的に定義できるオーケストレーションサービスです。今回のアップデートで、Amazon Bedrock AgentCore（AIエージェント実行基盤・プレビュー）との最適化統合により、**ワークフローの途中に「AIに考えさせるステップ」を差し込める**ようになりました。従来は Lambda を自作して LLM を呼んでいた部分を、マネージドなステップに置き換えられます。定型処理の中に「判断」を挟みたい運用（例: 障害内容を要約して分岐）に向きます。

### 1-2. 大規模ワークフローを支える実践ノウハウ（Contract One Entry 事例）
出典: <https://findy-tools.io/articles/sansan-aws/111>

Step Functions の状態管理に加えて、Amazon Aurora（マネージドDB）に処理状況を随時記録・更新することで整合性を担保している実例です。ポイントは **「二重実行の防止」と「安全な再実行（べき等性）」**。ワークフローが途中で失敗しても、DB 側の状態を見て「どこから再開すべきか」を判断できる設計にしておくと、運用が壊れにくくなります。自前バッチを Step Functions に載せ替える時の勘所として有用です。

### 1-3. 肥大化するステートマシンをどう設計するか（AWS Summit Japan 2026）
出典: <https://iret.media/200779>

「最初は単純だったワークフローが、関係者と機能の増加でステート数が膨張し、複雑なロジックが混入して開発生産性が落ちる」という"あるある"を正面から扱った設計論。AI の非決定性（同じ入力でも出力が揺れる性質）をワークフローでどう制御するか、という新しめの論点にも触れています。ワークフローを「大きな1枚」で作らず、部品化（ネストしたステートマシン）で分割する発想が学べます。

---

## 2. 直す — AWS Config で設定違反を「自動修復」する

### 2-1. 非準拠リソースを自動修復する機能（基本の型）
出典: <https://dev.classmethod.jp/articles/aws-config-auto-remediation/> / <https://dev.classmethod.jp/articles/automate-aws-config-remediation-action/>

AWS Config は、リソースの設定を継続的に記録・評価し「ルール違反（非準拠）」を検知するサービスです。この検知に **SSM Automation（Systems Manager の自動化ドキュメント）** を紐づけておくと、違反を見つけた瞬間に自動で直せます。例: 「S3 バケットが公開設定になっていたら自動で非公開に戻す」。棚卸しレポートを人が見て手で直す運用を、検知→修復まで一気通貫にできます。

### 2-2. CloudTrail が無効化されたら自動で再有効化（AWS 規範ガイダンス）
出典: <https://docs.aws.amazon.com/ja_jp/prescriptive-guidance/latest/patterns/automatically-re-enable-aws-cloudtrail-by-using-a-custom-remediation-rule-in-aws-config.html>

AWS 公式の規範ガイダンス（Prescriptive Guidance＝AWS 推奨の実装パターン集）。監査ログの要である CloudTrail が誰かに止められた場合、Config のカスタム修復ルールで**自動的に再有効化**する構成です。「守るべき設定を勝手に戻されない」ためのガードレールとして、そのまま流用できるパターンが提示されています。

### 2-3. 実装の落とし穴 — 自動修復ループと通知設計
出典: <https://blog.usize-tech.com/aws-config-auto-remediation-loop/> / <https://tech.nri-net.com/entry/security_notification_and_automatic_repair_function>

自動修復は便利な一方、**「直す→また検知される→また直す」の無限ループ**にハマることがあります。原因（評価タイミングや修復対象の指定ミス）と回避策を解説した実務記事と、修復と同時に通知（誰が・何を・いつ直したか）を組み込む設計のポイント記事。自動化は「暴走しない設計」とセットで、という教訓が詰まっています。

---

## 3. 守る — 脆弱性管理・脅威検知を自動化/AI化する

### 3-1. Amazon Inspector — ソフトウェア脆弱性を継続スキャン（Classic 廃止に注意）
出典: <https://aws.amazon.com/inspector/>

Inspector は EC2・ECR（コンテナレジストリ）・Lambda のソフトウェア脆弱性（CVE）とネットワーク露出を**継続的に自動スキャン**するサービス。エージェント配布や定期スキャンの手運用を不要にします。重要な移行情報として、旧世代の **Inspector Classic は 2026-05-20 に廃止済み**。まだ Classic 前提の運用が残っている場合は、現行 Inspector（自動有効化・自動スキャン型）への移行が必須です。

### 3-2. GuardDuty 調査エージェント — 脅威調査を「時間→分」に（AI活用）
出典: <https://aws.amazon.com/jp/blogs/news/introducing-the-amazon-guardduty-investigation-agent-on-demand-ai-powered-threat-assessment/>

GuardDuty は機械学習で不審な挙動（不正アクセス・マルウェア通信など）を検知する脅威検知サービス。新登場の **調査エージェント（プレビュー）** は、検知結果（Findings）を AI がオンデマンドで深掘りし、「本当に危険か・どこまで影響したか」の一次評価を自動生成します。従来は数時間かかっていた初動調査を数分に短縮。アラート疲れ（大量検知で人が処理しきれない状態）の緩和に効きます。

### 3-3. Inspector と GuardDuty の使い分け（役割の整理）
出典: <https://it-cert-lab.com/aws-curriculum-20260409/> / <https://medium.com/sudoconsultants/secure-your-aws-environment-with-guardduty-and-inspector-9ba56789dc9e>

混同しやすい2つを整理: **Inspector = 攻撃される前の「弱点」を見つける（脆弱性スキャン）**、**GuardDuty = 実際に起きている「攻撃・異常」を見つける（脅威検知）**。加えて Security Hub がそれらの結果を集約する、という三層構造です。2025〜2026年は GuardDuty の Extended Threat Detection（多段階攻撃シーケンスの検出）強化も進んでおり、両者を併用して「予防」と「検知」を自動でカバーするのが定石です。

---

## 4. 計画的な運用自動化 — EventBridge Scheduler × CloudWatch

### 4-1. メンテナンス時間だけ CloudWatch アラームを自動で抑制する
出典: <https://dev.classmethod.jp/articles/deisable-and-enable-cloudwatch-alarm-actions-amazon-eventbridge-scheduler/> / <https://dev.classmethod.jp/articles/amazon-eventbridge-scheduler-cloudformation-template-for-enable-disable-cloudwatchalarm/>

計画メンテナンス中は監視アラートが鳴って当然 —— でも夜間に叩き起こされたくない。**EventBridge Scheduler のユニバーサルターゲット**（任意の AWS API を時刻指定で叩ける機能）で、指定時間だけ CloudWatch アラームアクションを自動で無効化→終了後に自動で再有効化する構成です。CloudFormation テンプレート付きの記事もあり、**IaC でそのまま再現**できます。「メンテナンス台帳を見て手で止める」運用の卒業に。

### 4-2. スケジュール実行そのものを監視する
出典: <https://techblog.techfirm.co.jp/entry/scheduler-monitoring> / <https://blog.serverworks.co.jp/eventbridge_schedule-group>

自動化を増やすと今度は「その自動化がちゃんと動いたか」の監視が課題になります。EventBridge Scheduler のスケジュールグループ単位でメトリクスを集計し、**実行失敗を検知して通知**する方法の解説。自動化は「動かして終わり」ではなく「動き続けているかを見る」まで含めて設計する、というオブザーバビリティ（可観測性）の実践例です。

---

## 5. ベストプラクティス — AI と進める Well-Architected レビュー

### 5-1. AI と一緒に進める Well-Architected レビューのすすめ（AWS 公式）
出典: <https://aws.amazon.com/jp/blogs/news/aws-well-architected-review-with-ai-awssummit2026/>

Well-Architected Framework は AWS が長年の知見をまとめた設計ベストプラクティス集で、6本柱（運用の優秀性・セキュリティ・信頼性・パフォーマンス効率・コスト最適化・持続可能性）で構成されます。この公式ブログは、**レビュー作業そのものを AI で加速する**アプローチを紹介（AWS Summit Japan 2026 発表）。膨大なチェック項目に対する現状の棚卸しと改善提案の下書きを AI に作らせ、人がレビューする流れです。「運用の優秀性」柱は本連載のテーマそのもの（属人化の排除・障害対応の自動化）なので、自動化施策の"健康診断"として定期実行する価値があります。

---

## さらに試す価値（awesome シリーズ / リンク集）

手を動かす教材として、GitHub の awesome（厳選リンク集）シリーズは鉄板です。

- **awesome-aws**（総合・最も網羅的）: <https://github.com/donnemartin/awesome-aws>
- **awesome-aws-lambda**（サーバーレス自動化のネタ元）: <https://github.com/danteata/awesome-aws-lambda>
- **awesome-aws-security**（セキュリティ特化・CTF/実習含む）: <https://github.com/jassics/awesome-aws-security>
- **Awesome AWS Lists 横断検索**（トピック別に awesome を探せる）: <https://awesome.ecosyste.ms/lists?topic=aws>
- **Step Functions ユースケース集（AWS 公式）**: <https://docs.aws.amazon.com/ja_jp/step-functions/latest/dg/use-cases.html>

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
