# Hermes Agent × Discord 連携 構築手順（Windows 11 / WSL）

> ステータス: DONE / カテゴリ: SETUP
> 対象: Nous Research 製「Hermes Agent」を Windows 11 の WSL 上に構築し、Discord bot として接続するまでの手順書。
> 運用言語: 日本語（初心者にも分かる文体）。
> ⚠️ マスキング規約: ホスト名・ユーザー名・Discord ユーザーID 等は placeholder 化。**トークン等の秘密情報は一切記載しない**。実値は各自の手元（`~/.hermes/.env` 等）で管理する。

---

## 0. 概要

- **Hermes Agent** は Nous Research の OSS AI エージェント（Python 製）。`~/.hermes/` に状態を持つ。OpenClaw とは別製品。
- Discord 連携は **bot** として動作。DM またはサーバーチャンネルで対話できる。
- **アクセス制御は fail-closed**。OpenClaw のようなペアリングコード方式ではなく、`DISCORD_ALLOWED_USERS`（許可ユーザーID）で明示的に許可した人だけが使える。

### 前提

- Windows 11 ＋ WSL（Ubuntu 系ディストリビューション）。本手順は **すべて WSL 内のシェル**で実行する（PowerShell ではない）。
- Hermes Agent が WSL に導入済み（`hermes --version` が通る状態。例: v0.20.0）。
- モデルプロバイダの API キー設定済み（本構成では Anthropic）。

### 出典（公式・一次情報）

- Discord セットアップ: https://hermes-agent.nousresearch.com/docs/user-guide/messaging/discord/
- 設定（configuration）: https://hermes-agent.nousresearch.com/docs/user-guide/configuration/
- モデル設定（configuring models）: https://hermes-agent.nousresearch.com/docs/user-guide/configuring-models

---

## 1. Discord アプリ & bot の作成（ブラウザ）

1. [Discord Developer Portal](https://discord.com/developers/applications) を開き、**New Application** を作成（例: `<your-bot-name>`）。
   - **General Information の Application ID** をメモ（招待URLで使用）。
2. 左メニュー **Bot** を開く。
   - **公開Bot（Public Bot）= ON**（Discord 提供の招待リンクを使う場合に必要）。
   - **OAuth2 コードグラントが必要（Require OAuth2 Code Grant）= OFF**。

## 2. Privileged Gateway Intents（最重要）

Bot ページ下部 **Privileged Gateway Intents** で以下を **ON** にして **変更を保存**：

| Intent | 要否 | 備考 |
|---|---|---|
| Message Content Intent | **必須** | OFF だと「オンラインだが無反応」の典型原因（メッセージ本文が読めない） |
| Server Members Intent | **必須** | 許可ユーザーの解決に必要 |
| Presence Intent | 任意 | オンライン状態取得。不要なら OFF |

## 3. Bot トークンの取得

- Bot ページ **トークンをリセット（Reset Token）** → 表示された**トークンをコピー**。
- ⚠️ **一度しか表示されない**。パスワードマネージャ等に安全に保管。**チャットや Git に貼らない**。

## 4. 招待URLの生成とサーバー追加

- 手動URL（権限が確定済みで簡単）:

  ```
  https://discord.com/oauth2/authorize?client_id=<APP_ID>&scope=bot+applications.commands&permissions=274878286912
  ```

  - `<APP_ID>` は Step 1 の Application ID に置換。
  - `permissions=274878286912` に必要権限を内包（View Channels / Send Messages / Read Message History / Embed Links / Attach Files / Send Messages in Threads / Add Reactions）。
- URL をブラウザで開き、対象サーバーを選択して **認証（Authorize）**。CAPTCHA が出れば通す。
- サーバー招待後、bot と DM したい場合はサーバーの **Privacy Settings → Direct Messages を ON**。

## 5. 自分の Discord ユーザーID

- Discord 設定 → 詳細設定 → **開発者モード（Developer Mode）を ON**。
- 自分の名前を右クリック → **ユーザーIDをコピー**（`<your-discord-user-id>`）。許可ユーザーとして使う。

---

## 6. Hermes 側の設定（WSL）

対話セットアップを使うのが簡単：

```bash
hermes gateway setup
```

- プラットフォーム一覧で **↑↓ で Discord に合わせ ENTER/SPACE で選択**（一番下の Done を選ぶと未設定で終了するので注意）。
- プロンプトに従い入力：
  - **Discord bot token** → Step 3 でコピーしたトークン
  - **Allowed user IDs** → `<your-discord-user-id>`（カンマ区切りで複数可。空にすると誰でも可＝非推奨）
  - **Home channel ID** → 空 ENTER でスキップ可（後述 `/sethome` で設定可能）
- 続けて起動確認が出る：
  - **Start the gateway now? [Y/n]** → `Y`
  - **Start automatically on login/boot as a systemd service? [Y/n]** → `Y`
    - ユーザー systemd サービスが `~/.config/systemd/user/hermes-gateway.service` に導入され、**linger 有効化**でログアウト後も維持される。

### 手動設定（対話を使わない場合）

`~/.hermes/.env`（秘密は必ずこちら）:

```
DISCORD_BOT_TOKEN=<bot-token>
DISCORD_ALLOWED_USERS=<your-discord-user-id>
```

起動:

```bash
hermes gateway
```

---

## 7. 起動確認

```bash
hermes gateway status                          # active (running) を確認
journalctl --user -u hermes-gateway -f         # ログ追尾（Ctrl+C で終了）
```

- 数秒で Discord 上の bot がオンライン化。
- **DM＝メンション不要 / サーバーチャンネル＝@メンション必須**（既定）。bot に「テスト」と送って応答すれば成功。
- ログに `check_fn ... returned False`（image generation / browser / kanban 等）の **WARNING が出ても正常**。未設定のオプション機能が当ターンで使えないという通知に過ぎず、Discord チャットには影響しない。

### トラブルシュート

- **bot はオンラインだが無反応** → ほぼ **Message Content Intent が OFF**。Portal で ON にして保存し直す。
- **応答しない / 認可エラー** → `DISCORD_ALLOWED_USERS` に自分のIDが入っているか、bot が対象チャンネルを閲覧できるか確認。

---

## 8. Home Channel（通知の届け先）

cron 結果・リマインダー・通知を特定チャンネルに集めたい場合：

- **Home にしたいチャンネル（または DM）で `/sethome` を実行**（Discord チャット側のスラッシュコマンド。WSL ではない）。打った場所が Home になる。
- 代替: `~/.hermes/.env` に `DISCORD_HOME_CHANNEL=<channel-id>` を設定し、`hermes gateway restart`。

---

## 9. モデル設定（構成方針）

設定は `~/.hermes/config.yaml`（非秘密）と `~/.hermes/.env`（秘密）で管理。`hermes config` で操作する。反映は `hermes gateway restart`（既存セッションは次回新規から）。

本構成の方針:

- **メイン（デフォルト）= 最新 Sonnet**
- **軽作業（タイトル生成・要約圧縮・Web要約）= 最新 Haiku**（Sonnet で回すと割高なため）
- **Opus・Fable（いずれも最新）= `/model` で都度呼び出し可**
- **旧世代モデルは選択不可**（最新世代のみに限定）

### メイン = 最新 Sonnet

```bash
hermes config set model.provider anthropic
hermes config set model.default <latest-sonnet-model-id>
```

### 補助スロット（軽作業）= 最新 Haiku

```bash
hermes config set auxiliary.title_generation.provider anthropic
hermes config set auxiliary.title_generation.model <latest-haiku-model-id>
hermes config set auxiliary.compression.provider anthropic
hermes config set auxiliary.compression.model <latest-haiku-model-id>
hermes config set auxiliary.web_extract.provider anthropic
hermes config set auxiliary.web_extract.model <latest-haiku-model-id>
```

### `/model` で呼べるエイリアス（最新に固定）

```bash
hermes config set model.aliases.sonnet anthropic/<latest-sonnet-model-id>
hermes config set model.aliases.haiku  anthropic/<latest-haiku-model-id>
hermes config set model.aliases.opus   anthropic/<latest-opus-model-id>
hermes config set model.aliases.fable  anthropic/<latest-fable-model-id>
```

使い方: `/model opus`（そのセッションのみ）／`/model opus --once`（次の1ターンのみ）。

### 旧モデルを選択不可にする（最新世代のみ）

`hermes config edit` で `providers.anthropic` に列挙型を追記（リストは `hermes config set` では書けないため）：

```yaml
providers:
  anthropic:
    discover_models: false
    models:
      - <latest-opus-model-id>
      - <latest-sonnet-model-id>
      - <latest-haiku-model-id>
      - <latest-fable-model-id>
```

反映:

```bash
hermes gateway restart
```

- `hermes model` のピッカーに列挙した最新世代のみ表示されれば設定成功。
- モデルIDは提供状況が変わるため、実際の利用可否は `hermes model` のピッカーで確認して合わせる。

---

## 10. 常駐の堅牢化（WSL）

- `hermes gateway setup` の systemd サービス＋linger により、**ログアウト後も維持**。
- WSL 再起動耐性には `/etc/wsl.conf` に systemd 有効化が必要。本構成では設定済み：

  ```ini
  [boot]
  systemd=true
  ```

- これにより **Windows / WSL 起動時に systemd が上がり、Hermes サービスが自動復帰**する（WSL が起動している間だけ常駐、という性質は WSL の仕様）。
- Windows 起動と無関係な OS サービスにしたい場合は、root で `hermes gateway setup` を実行するオプションもある（案内メッセージ参照）。

---

## 11. 設定ファイルの所在（参考）

```
~/.hermes/
├── config.yaml   # 非秘密設定（model, terminal, auxiliary 等）
├── .env          # 秘密（API キー・bot トークン等）
├── auth.json     # OAuth 資格情報
├── SOUL.md       # エージェントの人格
├── memories/     # 永続メモリ（MEMORY.md, USER.md）
├── cron/         # 定期ジョブ
├── sessions/     # セッション
└── logs/         # ログ（秘密は自動マスク）
```

- 秘密は必ず `.env`。非秘密は `config.yaml`。両方に同じキーがある場合、非秘密は `config.yaml` が優先。
- 実行環境: `terminal.backend`（既定 `local` はエージェントがユーザーのファイルにフルアクセス。隔離したい場合は `docker` を検討）。

---

## 付録: 完了状態（本構築時点）

- Discord bot 接続: 成功（メンションに応答を確認）。
- 常駐: ユーザー systemd サービス＋linger 有効／`/etc/wsl.conf` に `systemd=true`。
- メインモデル: 最新 Sonnet（Anthropic）。
- 補助・エイリアス・旧モデル制限・Home Channel: 上記手順に沿って設定。
