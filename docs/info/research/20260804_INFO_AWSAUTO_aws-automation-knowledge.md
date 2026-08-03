作成日: 2026-08-04 / STATUS: INFO / TOPIC: AWSAUTO

# AWS 業務自動化ナレッジ（週次）— AIOps で「調べる・気づく・直す」を自動化する

## 今週のテーマ / 見どころ

先週まではワークフロー（Step Functions）や設定違反の自動修復、鍵レス CI/CD を扱いました。今週は視点を変えて **AIOps（= AI for IT Operations、AI を使った IT 運用の自動化・省力化）** を横串テーマにします。2026 年の AWS は「人が画面を睨んで原因を探す」作業を AI に肩代わりさせる方向へ大きく舵を切っており、オブザーバビリティ（可観測性＝システムの内部状態をログ・メトリクス・トレースから把握できること）、コスト管理、障害の先読み、定型オペレーションのそれぞれで "AI が一次調査を済ませてから人が判断する" 形が現実になってきました。

今週の主役は次の4つです。

- **見る × AI**: CloudWatch / Application Signals の MCP サーバーで、自然言語のまま障害調査ができる。
- **コスト × AI**: Cost Anomaly Detection に「AI コスト調査」、さらに FinOps 専用エージェントが登場。
- **気づく × AI**: DevOps Guru が機械学習で異常を"障害になる前"に検知。
- **直す（定型）**: SSM Automation ランブックと AWS Backup で、手作業のオペレーションを機械に渡す。

初心者向けに用語を先に整理しておきます。**MCP（Model Context Protocol）** は、AI エージェントに外部ツール（今回は監視データ）を安全に接続するための共通規格です。**SLO（Service Level Objective、サービス品質目標）** は「レイテンシは99パーセンタイルで300ms 以内」のような運用上の約束事です。

---

## 1. 見る × AI — オブザーバビリティを「自然言語で調べる」時代へ

### 1-1. CloudWatch と Application Signals の MCP サーバー（AIOps の本命）
出典: [Enhance your AIOps: CloudWatch と Application Signals MCP サーバー（AWS 公式ブログ）](https://aws.amazon.com/jp/blogs/news/enhance-your-aiops-introducing-amazon-cloudwatch-and-application-signals-mcp-servers/) / [AWS Labs MCP: CloudWatch MCP Server](https://awslabs.github.io/mcp/servers/cloudwatch-mcp-server) / [同: Application Signals MCP Server](https://awslabs.github.io/mcp/servers/cloudwatch-applicationsignals-mcp-server)

AWS がオープンソースで公開した2つの MCP サーバーです。**CloudWatch MCP サーバー**はアラームを起点にしたインシデント対応、メトリクス分析、ログのパターン検出を担当。**Application Signals MCP サーバー**は SLO に基づくサービス正常性の監視と、OpenTelemetry（オープンな計装規格）データを使った自動根本原因分析を担当します。ポイントは、AI エージェントに対して「このアラーム、何が原因？」と**自然言語で聞くだけ**で、アラームパターン・メトリクス異常・ログ・トレースを横断して調査してくれること。監視ダッシュボードを何枚も切り替える作業が丸ごと不要になります。

### 1-2. GitHub Action 化で「開発フローの中」に観測を組み込む
出典: [Application Signals に GitHub Action と MCP サーバー改善が追加（AWS What's New, 2025-11）](https://aws.amazon.com/about-aws/whats-new/2025/11/amazon-cloudwatch-application-signals-adds-github-action-mcp-server-improvements/)

Application Signals の MCP サーバーは AI コーディングエージェント（Kiro など）からも使えるようになり、「レイテンシ悪化・エラー・SLO 違反の原因になっている**正確なファイル・関数・行**」まで特定できるようになりました。さらに GitHub Action が一般提供（GA）され、アプリの可観測性を開発者ツールの中に持ち込めます。**運用と開発の間の壁**（本番で見えた劣化を、コードのどこが原因か突き止める作業）をここまで縮めた点が実務的です。注意点として、MCP サーバー経由での調査は読み取り権限を最小化した専用ロールで動かすのが安全です。

### 1-3. まず何が見えるのか — 導入検証レポート
出典: [Application Signals を導入してなにが見えるか試してみた（サーバーワークス）](https://blog.serverworks.co.jp/amazon-cloudwatch-application-signals-trial) / [2026年1〜5月の AWS オブザーバビリティ関連リリースまとめ（AWS 公式）](https://aws.amazon.com/jp/blogs/news/aws-observability-icymi-jan-may-2026/)

「MCP や AI の前に、そもそも Application Signals で何が見えるのか」を掴むための検証記事。サービス同士の関係を自動で可視化し、リクエスト率・レイテンシ・エラー率といった "ゴールデンメトリクス"（アプリの健康状態を示す基本指標）を最小設定で収集できる様子が分かります。公式の ICYMI（= In Case You Missed It、見逃し向けまとめ）によると、2026年前半だけで CloudWatch / X-Ray / Managed Grafana / Managed Prometheus に40件超のアップデートがあり、テーマは「OpenTelemetry 対応強化」と「AI 駆動の運用」の2つ。**まず Application Signals を有効化 → 慣れたら MCP で AI 調査**という段階導入が現実的です。

---

## 2. コスト × AI — 異常検知から「原因調査」まで自動で

### 2-1. Cost Anomaly Detection に「AI コスト調査」が追加
出典: [AWS now provides AI-powered cost investigations（AWS What's New, 2026-06）](https://aws.amazon.com/about-aws/whats-new/2026/06/aws-ai-powered-cost-investigations/) / [【概要のみ紹介】Cost Anomaly Detection に Amazon Q の AIコスト調査（サーバーワークス）](https://blog.serverworks.co.jp/2026/06/16/175035)

Cost Anomaly Detection（コストの急増を自動検知する機能）に、Amazon Q による**根本原因の自動調査**が付きました。従来は「先週より請求が跳ねた」というアラートまでで、原因究明は人手でした。今回の AI 調査は、**使用量が増えたのか単価が上がったのか**を判別し、寄与したサービス・アカウント・リージョンを特定。使用量起因の場合は CloudTrail（API 操作の監査ログ）と突き合わせて「どの API 呼び出し・どの IAM プリンシパル（利用者）が原因か」まで数分で平文の説明にしてくれます。FinOps（クラウド財務管理）担当がアラートから対処へ動くまでの時間を大幅短縮できます。

### 2-2. AWS FinOps Agent（パブリックプレビュー）
出典: [AWS FinOps Agent（AWS 公式）](https://aws.amazon.com/finops-agent/) / [AIが教える「AWS FinOps Agent」プレビュー開始（ITmedia）](https://www.itmedia.co.jp/news/articles/2606/30/news097.html)

さらに一歩進んだ、クラウド財務管理専用のエージェント型 AI。コスト異常を根本原因まで掘り下げ、エンジニアが普段使う **Jira や Slack の中で**コストの質問に答えます。異常を CloudTrail イベントと相関させて「原因になった変更」と「担当オーナー」を割り出し、調査サマリを自動生成。オプションで Jira チケット起票や Slack 投稿まで実行するので、リソースの持ち主に文脈ごと通知が飛びます。**「コストの番人」を常駐させる**イメージで、棚卸し会議の前準備が要らなくなる方向性です。プレビュー段階のため、まずは非本番アカウントで挙動と権限範囲を確認してから広げるのが無難です。

---

## 3. 気づく × AI — 障害になる"前"に検知する

### 3-1. Amazon DevOps Guru で異常を先読み
出典: [DevOps Guru のログ記録とモニタリング（AWS 公式ドキュメント）](https://docs.aws.amazon.com/ja_jp/devops-guru/latest/userguide/monitoring-overview.html) / [AWS上のアプリ障害監視をAIに任せる（DevOps Guru + Chatbot カスタム通知・Zenn）](https://zenn.dev/ncdc/articles/d887da754389c2)

DevOps Guru は、CloudWatch・Config・CloudTrail・CloudFormation・X-Ray のメトリクスやイベントを機械学習で学習し、**異常な振る舞いを障害化する前に検知**して推奨対処まで提示するサービスです。強みは「手動でしきい値アラームを組んでいない想定外の劣化」も拾えること。有効化するだけで学習が始まる手軽さがあります。Zenn の事例では DevOps Guru の検知を **Amazon Q Developer in chat applications（旧 AWS Chatbot）** 経由で Slack へカスタム通知に整形しており、"AI が気づく → チャットに要点だけ流れてくる" 運用の型が参考になります。コストは監視対象リソース量に応じて増えるため、重要サービスから段階適用するのがコツです。

### 3-2. SSM Automation ランブックで定型オペを機械に渡す
出典: [Systems Manager Automation で始める運用自動化（クロスパワー, 2026-02）](https://xp-cloud.jp/blog/2026/02/26/100000) / [Automation で運用保守作業を自動化しよう（iret.media）](https://iret.media/192176) / [AWS Systems Manager Automation（AWS 公式ドキュメント）](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-automation.html)

異常に「気づいた後」の定型対応は、SSM（Systems Manager）Automation の**ランブック**（= 手順を YAML/JSON で定義した実行可能な運用マニュアル）に落とし込めます。AWS が用意する定義済みランブック（インスタンスタイプ変更、再起動、パッチ適用等）をそのまま使うか、独自の複数ステップ手順を組めます。EventBridge と組み合わせれば「特定イベント → ランブック自動実行」のイベント駆動運用に。障害対応の初動を人手からコードへ移すことで、平均復旧時間（MTTR）の短縮と対応の属人化解消に直結します。まずは"再起動""スナップショット取得"など低リスクな手順から自動化するのが定石です。

---

## 4. 守る（データ保護）— AWS Backup で世代管理を自動化

出典: [AWS Backup によるデータレイク保護のベストプラクティス（AWS 公式ブログ）](https://aws.amazon.com/jp/blogs/news/best-practices-for-data-lake-protection-with-aws-backup/) / [AWS Backupとは？主要機能・運用ベストプラクティス（GMO）](https://managed.gmocloud.com/library/aws/operations/aws-backup-best-practices.html)

バックアップは「面倒だが止められない」典型的ルーティンワーク。AWS Backup はバックアッププランで **Cron 式**によるスケジュール、保持期間・自動削除、リージョン間／アカウント間コピーを一元管理でき、EC2・RDS・S3 などを横断してポリシー適用できます。ベストプラクティスの肝は **Backup Vault（保管庫）の Vault Lock と IAM ポリシー**で、誤削除・不正アクセス・ランサムウェアから世代を守ること（変更不可の保持設定）。公式のデータレイク保護記事は、S3 上の重要データにきめ細かなアクセス制御と監査を効かせる構成を示しています。「取れているつもり」を避けるため、復元テストの定期自動化（SSM Automation でリストア検証）まで含めて設計すると安心です。

---

## 5. さらに試す価値（awesome シリーズ / リンク集）

- [donnemartin/awesome-aws](https://github.com/donnemartin/awesome-aws) — 定番の AWS まとめ。ライブラリ・OSS・ガイドを網羅した出発点。
- [Noovolari/awesome-cloudops](https://github.com/Noovolari/awesome-cloudops) — CloudOps（クラウド運用）に絞ったツール＆ベストプラクティス集。今週テーマと親和性が高い。
- [DNXLabs/awesome-aws-bedrock](https://github.com/DNXLabs/awesome-aws-bedrock) — Bedrock（生成 AI 基盤）関連。自作の AIOps エージェントを組むときの部品探しに。
- [不要リソースを Lambda + EventBridge で自動検知（Zenn）](https://zenn.dev/team_delta/articles/3df947b2fca982) — 削除忘れの EBS など高額な"置き忘れ"を検知する、コスト棚卸し自動化の実装例。今週の FinOps テーマの手を動かす版として。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
