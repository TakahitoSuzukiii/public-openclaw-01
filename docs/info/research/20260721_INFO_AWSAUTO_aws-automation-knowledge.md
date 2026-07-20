# AWS 業務自動化ナレッジ（週次）— 生成AI運用調査・鍵レスCI/CD・SSM実務

作成日: 2026-07-21 / STATUS: INFO / TOPIC: AWSAUTO

## 今週のテーマ / 見どころ

今週は「人手を減らす運用」を **3つの軸** で集めました。

1. **生成AIに運用を任せる** — Amazon Q Developer で障害調査（根本原因の推測）を高速化。属人化（特定の人しか対応できない状態）の解消につながります。
2. **鍵を持たないデプロイ** — GitHub Actions × OIDC（OpenID Connect＝ID連携の標準規格）で、長期のアクセスキーを廃止する「鍵レス（keyless）」CI/CD。漏えい事故そのものを設計で消します。
3. **SSM（Systems Manager）で運用の8割を自動化** — Session Manager / Run Command / Patch Manager / Automation の実務的な使い分け。

先週（07-14）はコストレポート自動化と自動修復（ASR）中心でした。今週は重複を避け、**AI運用・CI/CD・脆弱性CI連携** に寄せています。

用語の下ごしらえ:
- **IaC** = Infrastructure as Code = インフラ構成をコードで管理する考え方。
- **CI/CD** = Continuous Integration / Continuous Delivery = 継続的インテグレーション/継続的デリバリー。テストとデプロイを自動化する仕組み。
- **SRE** = Site Reliability Engineering = 信頼性をエンジニアリングで担保する運用手法。
- **CVE** = Common Vulnerabilities and Exposures = 既知の脆弱性に付く共通の識別番号。

---

## 1. 生成AIで運用調査・障害対応を自動化する

### 1-1. Amazon Q Developer で「運用上の問題」を調査・修正（プレビュー）
出典: <https://aws.amazon.com/jp/blogs/news/investigate-and-remediate-operational-issues-with-amazon-q-developer/>

CloudWatch のアラームや CloudTrail（API 操作の監査ログ）を Amazon Q Developer が横断的に読み、「何が起きて、根本原因は何か」の**仮説を自動生成**してくれる新機能です（米国東部リージョンでプレビュー）。障害時に人間が複数コンソールを行き来していた一次調査を AI が肩代わりします。リソースにタグを付けておくと関連リソースの検出精度が上がるのがコツ。まずは検証環境で「調査の下書き」を作らせる用途から始めると安全です。

### 1-2. Amazon Q Developer による運用管理の実像（インシデント調査の高速化）
出典: <https://biz.nuro.jp/column/aws-mama-092/>

上記機能を運用者目線でかみ砕いた解説記事。CloudWatch との連携で調査を迅速化し、**属人化の解消**（＝ベテラン依存からの脱却）に効く点を強調しています。「AI に全部任せる」のではなく、AI が出した仮説を人間がレビューする HITL（Human-in-the-Loop＝人が判断ループに入る）運用が現実的、という温度感が参考になります。

### 1-3. 実際に障害調査を試してみた検証レポート
出典: <https://zenn.dev/nnydtmg/articles/aws-amazon-q-operational-investigation>

個人検証ベースで Amazon Q Developer の運用調査を触った記事。どの画面から起動し、どんな出力が返るかの一次体験が読めます。公式の建付けだけでは分からない「実際の当たり外れ」を掴むのに有用です。

---

## 2. 鍵レス CI/CD — OIDC で長期アクセスキーを捨てる

### 2-1. GitHub 公式: Actions から AWS へ OIDC で接続
出典: <https://docs.github.com/ja/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws>

GitHub Actions が発行する短命の OIDC トークンを AWS が信頼し、**アクセスキーを Secrets に保存せずに** IAM ロールを引き受けてデプロイする方式の一次情報。長期クレデンシャルの棚卸し・ローテーションという「面倒なルーティン」ごと消せるのが最大の利点です。まずは信頼ポリシーの `sub`（対象リポジトリ/ブランチ）を絞る設計から。

### 2-2. OIDC 鍵レス本番デプロイの実装プレイブック
出典: <https://www.ai2core.com/posts/2026-03-05-github-actions-oidc-secure-deploy-playbook/>

設計・実装・**監査**までを一連の手順書としてまとめた記事（2026-03）。「漏えい事故を減らす」観点で、ロールの権限最小化やブランチ限定の条件付けを具体化しています。セキュリティレビューの観点付きなので、本番導入前のチェックリストとして使えます。

### 2-3. Terraform / CDK / Lambda への OIDC 適用例
出典（Terraform）: <https://qiita.com/tanaka_meiya/items/efeb4ad5e9de9e3874c3>
出典（CDK・クラスメソッド）: <https://dev.classmethod.jp/articles/github-actions-oidc-aws-cdk-deploy/>

IaC ツール別の適用ハンズオン。CDK 版は「main への push で自動デプロイ」まで通す構成、Terraform 版は OIDC Provider の作成から解説。手動 `cdk deploy` / `terraform apply` の押し忘れ・打ち間違いを無くす即効ネタです。

---

## 3. SSM（Systems Manager）で運用の“定型作業”を自動化

### 3-1. SSM Automation で複数ステップの運用を自動化
出典: <https://xp-cloud.jp/blog/2026/02/26/100000>

定期作業や障害対応手順を **Runbook（ランブック＝手順書をコード化したもの）** にして、複数ステップのワークフローとして自動実行する入門記事。「手順書の Word を見ながら人が実行」を卒業できます。まずは既存の運用手順を1本 Runbook 化するのが着手点。

### 3-2. Session Manager / Run Command / Patch Manager の使い分け
出典: <https://infra-academy.net/aws-systems-manager-ec2-automation/>

SSH を開けずに（＝踏み台や 22 番ポートなしで）EC2 を操作する Session Manager、複数台へ一括コマンドの Run Command、OS パッチ適用の Patch Manager を整理。**攻撃面（開放ポート）を減らしつつ運用を自動化**という、セキュリティと運用効率を両立する定番構成です。

### 3-3. セキュリティパッチ適用の自動化（Patch Manager 実務）
出典: <https://www.bes-pra.net/manual/automated-patching.html>

なぜパッチ自動化が要るか、Patch Manager の仕組み、**運用時の落とし穴**（再起動タイミング、パッチベースライン、メンテナンスウィンドウ）までを平易に解説。ゼロデイ（未修正の新規脆弱性）対応の初動を速くする土台になります。

---

## 4. セキュリティ / 脆弱性管理の自動化

### 4-1. Amazon Inspector × Systems Manager で検出から修復まで（公式）
出典: <https://aws.amazon.com/jp/blogs/news/automate-vulnerability-management-and-remediation-in-aws-using-amazon-inspector-and-aws-systems-manager-part-1/>

Inspector が CVE を継続検出 →  Security Hub のカスタムアクション → SSM Automation で EC2 を**ワンクリック（or 自動）修復**する公式ハンズオン。検出→チケット→手作業という往復を短絡できます。

### 4-2. 「疲弊しない」脆弱性管理の設計思想
出典: <https://iret.media/190424>

Inspector を「攻撃検証ではなく**事実把握**（既知 CVE の継続検出）」と位置づけ、アラート洪水で運用が疲弊しないための優先度設計を論じた良記事。**自動化の前に“何を無視してよいか”を決める**という運用設計の勘所が得られます。

---

## 5. さらに試す価値（ベストプラクティス / リンク集）

- **AI と一緒に進める Well-Architected レビュー（AWS Summit 2026）** — 潜在リスクを体系的に炙り出す公式ガイド: <https://aws.amazon.com/jp/blogs/news/aws-well-architected-review-with-ai-awssummit2026/>
- **運用上の優秀性の柱（公式ホワイトペーパー）** — 「運用をコードとして実行する」の原典: <https://docs.aws.amazon.com/ja_jp/wellarchitected/latest/operational-excellence-pillar/welcome.html>
- **AWS 利用料を Lambda + EventBridge Scheduler で自動監視・Slack 通知** — 前月比・日次トレンド・Cost Anomaly Detection まで: <https://blog.serverworks.co.jp/notify-aws-cost-using-lambda-and-eventbridge>
- **EventBridge × Lambda で「平日・非祝日だけ実行」** — 起動/停止スケジュールの応用パターン: <https://zenn.dev/nari_rina/articles/e87fb18572ff35>
- **awesome-aws（GitHub のキュレーション集）** — ライブラリ/ツールの定番まとめ: <https://github.com/donnemartin/awesome-aws>

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
