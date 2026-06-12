# Playwright MCP 活用ガイド — おすすめの使い方とヘッドレス実演

> 作成日: 2026-06-12 / STATUS: INFO / TOPIC: PLAYWRIGHT
> 目的: ブラウザ自動化「Playwright」と、それをAIから使う「Playwright MCP」のおすすめの使い方を初心者にも分かるように整理し、本サーバ（OpenClaw）でのヘッドレス実演結果をまとめる。
> 関連: 人気MCP/Skills `docs/info/research/20260612_INFO_MCP_popular-mcp-and-skills-for-openclaw.md`、導入手順 `docs/openclaw/006_DONE_SETUP_playwright-mcp-setup.md` / `007_DONE_SETUP_playwright-browser-setup.md`。
> 注意: スクレイピング等の自動取得は対象サイトの利用規約・robots.txt・API提供有無を必ず確認すること（本サーバの運用ルール）。

---

## 1. Playwright と Playwright MCP とは

- **Playwright**: Microsoft 製のブラウザ自動化フレームワーク。Chromium/Firefox/WebKit を操作でき、**自動待機（auto-waiting）** が強力で「要素が出るまで待つ」を勝手にやってくれるため、テストが安定します。
- **Playwright MCP**: その機能を **MCP（AI共通プラグイン規格）** 経由でAIに渡すサーバ。AIが「ページを開く・クリック・入力・スクショ・抽出」を*ツール*として呼べます。
- 最大の特徴: 操作の起点が**ピクセル画像ではなく「アクセシビリティ・スナップショット（ARIA ツリー）」**。AIは構造化テキストで要素を把握して操作するため、速く・安定し・トークンも節約できます（出典: anhtu.dev / dataflirt）。

---

## 2. おすすめの使い方（カテゴリ別）

### A. UI / E2E テスト（最も人気）
- 画面を操作させて**テストを自動生成**、壊れたテストを**自己修復（self-healing）**、Selenium からの移行支援（出典: Testleaf / frugaltesting）。
- 自動待機＋Webファースト・アサーションで「待ち」のバグが激減。

### B. Web スクレイピング / データ抽出
- アクセシビリティ・スナップショットを使い、LLM が「見出し・リンク・本文」を構造的に抽出（出典: dataflirt）。
- ※ 利用規約・robots.txt の順守が前提（本サーバの必須ルール）。

### C. UI 検証・スクリーンショット
- デプロイ後の見た目確認、リグレッション検出、デザインレビュー用のスクショ取得。
- 本サーバでも #49/#50 のダッシュボード改修時に、この方法で描画検証を実施。

### D. ログイン/認証フロー・フォーム操作
- ログイン状態の確認、フォーム自動入力、複数タブ操作。
- セッション/Cookie を保持して「ログイン後だけ見える情報」の確認にも。

### E. ネットワーク監視・デバッグ
- リクエスト/レスポンスの取得、コンソールエラーの収集（#49/#50 でも favicon 404 等を確認）。

---

## 3. ヘッドレスモード実演（本サーバで実行）

「ブラウザのヘッドレスモードで何かやってみたい」という依頼に対し、本サーバの Playwright MCP で**ヘッドレス実行**を実演しました。

**手順と結果:**
1. `https://example.com` へナビゲート（ヘッドレス・画面なし）。
2. **スクリーンショット取得** → `tests/playwright-demo/playwright-mcp-demo-example.png`（17KB）。
3. **アクセシビリティ・スナップショット取得**（操作の基盤となる構造化データ）:
   ```yaml
   - heading "Example Domain" [level=1]
   - paragraph: This domain is for use in documentation examples without needing permission. Avoid use in operations.
   - paragraph:
     - link "Learn more": https://iana.org/domains/example
   ```

→ 画像（スクショ）と構造（スナップショット）の**両方**を取得できることを確認。AIは後者（ARIAツリー）を見て「『Learn more』リンクをクリック」のように*要素名で*操作できます。これがピクセル座標頼みの自動化より堅牢な理由です。

---

## 4. OpenClaw での実用アイデア

1. **ダッシュボード/サイトの定期スクショ監視**: 重要画面を定期取得し、見た目崩れを検知。
2. **ログイン確認の定常化**: 認証が切れていないかを定期チェックして通知。
3. **公開情報の構造抽出**（規約順守の範囲で）: 公式リリースノートや価格表など、APIが無いページをスナップショット経由で要約。
4. **改修時の自動描画検証**: フロント変更後にヘッドレスでスクショ＋コンソールエラー確認（本セッションで実践済み）。

> 既存構成: 本サーバは Playwright MCP / ブラウザ導入済み（`docs/openclaw/006`・`007`）。ヘッドレス前提で運用。日本語フォントが無い環境ではスクショ上で日本語が空白になるため、文字内容の検証は**スナップショット（テキスト）**を併用するのが確実（本セッションで確認済みの注意点）。

---

## 5. まとめ
- Playwright MCP の肝は「**スナップショット（ARIA）主体の操作**」。速い・安定・低トークン。
- 用途は **E2Eテスト > スクレイピング > UI検証 > 認証/フォーム > ネットワーク監視** の順で人気。
- 本サーバでは**ヘッドレス実演に成功**（example.com のスクショ＋スナップショット取得）。次は「自社ダッシュボードの定期スクショ監視」など小さな常設タスクから始めるのがおすすめ。ご希望あればタスク化します。

---

## 出典（2026年時点・参考）
- anhtu.dev「Playwright 2026 — E2E Testing, MCP, AI-Assisted Browser Automation」: [https://anhtu.dev/playwright-2026-e2e-testing-mcp-ai-assisted-browser-automation-1116](https://anhtu.dev/playwright-2026-e2e-testing-mcp-ai-assisted-browser-automation-1116)
- dataflirt「Playwright MCP Guide: Web Scraping, Testing, more」: [https://dataflirt.com/blog/playwright-mcp-guide/](https://dataflirt.com/blog/playwright-mcp-guide/)
- Testleaf「Practical Use Cases of Playwright MCP in 2026」: [https://www.testleaf.com/blog/web-stories/practical-use-cases-of-playwright-mcp-in-2026/](https://www.testleaf.com/blog/web-stories/practical-use-cases-of-playwright-mcp-in-2026/)
