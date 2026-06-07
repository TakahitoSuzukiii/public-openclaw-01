# フォールバック発生通知サービス 構築手順

> STATUS: DONE / CATEGORY: SETUP / 作成日: 2026-06-07
> primary(Claude) が応答不可となり fallback モデル（Gemini 等）へ自動切替された瞬間を検知し、Discord(NEXUS チャット) へ即時通知する常駐サービスの構築手順です。010（Gemini フォールバック）の続き。

## 0. 概要（何を実現するか）

[010](./010_DONE_SETUP_gemini-fallback.md) で Claude → Gemini の自動フォールバックを構成しました。本サービスは、その**フォールバックが実際に発火したことを鈴木さんへ即座に知らせる**ものです。

- gateway のログには、フォールバック判断が機械可読な JSON イベントで記録される。
- このログを常時追従（follow）し、「primary → fallback へ切替」イベントを検知したら Discord へ通知。
- 連続発火（Claude 障害が続くと毎リクエストで切替）によるスパムを防ぐため**スロットリング**（既定 5 分に 1 回）。

> 用語: **常駐サービス（daemon）** … バックグラウンドで動き続けるプログラム。ここでは systemd user service として登録し、サーバ起動時・プロセス異常終了時も自動で立ち上がる。
> 用語: **スロットリング（throttling）** … 短時間に大量発生するイベントの通知頻度を制限すること。

## 1. アーキテクチャ

```mermaid
flowchart TD
    GW["OpenClaw Gateway"] -->|フォールバック判断を記録| LOG["gateway ログ<br/>(model_fallback_decision)"]
    SVC["openclaw-fallback-notify.service<br/>(systemd user)"] -->|openclaw logs --follow --json| LOG
    SVC -->|next_fallback 検知| THROTTLE{"前回通知から<br/>5分以上?"}
    THROTTLE -->|Yes| SEND["openclaw message send<br/>--channel discord"]
    THROTTLE -->|No (抑制)| SKIP["スキップ"]
    SEND --> DISCORD["Discord (NEXUS チャット)"]
```

## 2. 検知するログイベント

gateway ログ（`openclaw logs --json`）に出る以下の行を検知対象とする。

```json
{"event":"model_fallback_decision", ...,
 "fallbackStepFromModel":"anthropic/claude-opus-4-8",
 "fallbackStepToModel":"google/gemini-2.5-flash",
 "fallbackStepFinalOutcome":"next_fallback",
 "nextCandidateProvider":"google",
 "nextCandidateModel":"gemini-2.5-flash",
 "fallbackConfigured":true}
```

- 判定キー: 行内に `model_fallback_decision` かつ `next_fallback` を含むこと（= primary 失敗 → 次の fallback へ切替が確定した瞬間）。
- 通知に使う値: `nextCandidateProvider` / `nextCandidateModel`（切替先モデル）。

> `candidate_failed` だけの中間行や、fallback 未設定時の `chain_exhausted` 行は対象外。実際に「fallback へ切り替えた」行のみ拾う。

## 3. 構成ファイル

### 3.1 通知スクリプト

`~/.openclaw/scripts/fallback-notify.sh`（実行権限付与: `chmod +x`）

要点:
- `openclaw logs --follow --json --no-color` でログを追従し、`while read` で 1 行ずつ評価。
- 判定行で `nextCandidateModel` / `nextCandidateProvider` を抽出。
- `notify()` がスロットル（`~/.openclaw/state/fallback-notify.last` に最終通知の epoch を保存）を見て、`FALLBACK_NOTIFY_THROTTLE_SEC`（既定 300 秒）未満なら抑制、それ以外は送信。
- 送信: `openclaw message send --channel discord --target user:<DISCORD_USER_ID> -m "..."`。
- ログ追従プロセスが落ちても systemd が再起動するため、スクリプト自体は単純な追従ループに徹する。

> 通知先 `--target` は `user:<DISCORD_USER_ID>` 形式。実値（ユーザ ID）はスクリプト内に記載（機密ではないが、本ドキュメントでは `<DISCORD_USER_ID>` でマスク）。

### 3.2 systemd user サービス

`~/.config/systemd/user/openclaw-fallback-notify.service`

```ini
[Unit]
Description=OpenClaw fallback notifier (Claude primary -> fallback model alerts to Discord)
After=openclaw-gateway.service
Wants=openclaw-gateway.service

[Service]
Type=simple
Environment=PATH=/usr/local/bin:/usr/bin:/bin:/home/<your-user>/.npm-global/bin
Environment=FALLBACK_NOTIFY_THROTTLE_SEC=300
ExecStart=/home/<your-user>/.openclaw/scripts/fallback-notify.sh
Restart=always
RestartSec=10

[Install]
WantedBy=default.target
```

- `node` は `/usr/bin/node`。openclaw の shebang が `#!/usr/bin/env node` のため、`PATH` に `/usr/bin` を含める。
- gateway と同じ systemd **user** サービス。`loginctl` の `Linger=yes` によりログイン無しでも常駐。

## 4. 導入コマンド

```bash
chmod +x ~/.openclaw/scripts/fallback-notify.sh
systemctl --user daemon-reload
systemctl --user enable --now openclaw-fallback-notify.service
systemctl --user status openclaw-fallback-notify.service   # active (running) を確認
```

## 5. 検証（2026-06-07 実施）

| 項目 | 方法 | 結果 |
|---|---|---|
| サービス稼働 | `systemctl --user status` | OK active (running)、`logs --follow` 子プロセス起動 |
| 検知ロジック | 実ログの `next_fallback` 行を抽出処理に通す | OK `google/gemini-2.5-flash` を正しく抽出 |
| 通知到達 | `openclaw message send` でテスト送信 | OK Discord 受信（Message ID 取得） |

> 参考: 2026-06-07 12:21(UTC) に実際のフォールバックが 1 度発火していた（"Claude CLI failed" → `google/gemini-2.5-flash`）。本サービスはそれ以降の発火を捕捉する。

## 6. 運用・調整

- **スロットル間隔の変更**: サービスの `FALLBACK_NOTIFY_THROTTLE_SEC` を編集 → `systemctl --user daemon-reload && systemctl --user restart openclaw-fallback-notify.service`。
- **停止 / 無効化**: `systemctl --user disable --now openclaw-fallback-notify.service`。
- **ログ確認**: `journalctl --user -u openclaw-fallback-notify.service -f`。
- **状態リセット**: `rm ~/.openclaw/state/fallback-notify.last`（次回イベントで即通知）。

## 7. 既知の制約・今後

- gateway 再起動の瞬間に発生したイベントは、追従プロセス再接続までの極短時間で取りこぼす可能性がある（systemd 再起動は数秒）。
- `chain_exhausted`（fallback も失敗 = 全滅）は現状未通知。必要なら別メッセージとして追加可能。

## 8. 関連

- 010_DONE_SETUP_gemini-fallback — フォールバック本体の構成
- ログ CLI: <https://docs.openclaw.ai/cli/logs>
- メッセージ送信 CLI: `openclaw message send`
