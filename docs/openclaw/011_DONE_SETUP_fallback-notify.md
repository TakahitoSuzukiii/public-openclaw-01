# フォールバック発生通知サービス 構築手順

> STATUS: DONE / CATEGORY: SETUP / 作成日: 2026-06-07
> primary(Claude) が応答不可となった際のモデルフォールバック関連イベントを検知し、Discord(NEXUS チャット) へ即時通知する常駐サービスの構築手順です。[[010_DONE_SETUP_gemini-fallback]] の続き。

## 0. 概要（何を実現するか）

[010](./010_DONE_SETUP_gemini-fallback.md) で Claude → Gemini の自動フォールバックを構成しました。本サービスは、その状況を鈴木さんへ即座に知らせます。**2 種類の通知**を明確に分けて送ります。

| 種別 | 発生条件 | 通知の意味 |
|---|---|---|
| **切替通知** ⚠️ | primary(Claude) 失敗 → fallback(Gemini 等) へ切替 | 会話は継続。Claude が一時的に不調 |
| **全滅通知** 🛑 | Claude も fallback も全て失敗 | 自動復旧できず。手動確認を推奨 |

- gateway ログにはフォールバック判断が機械可読な JSON イベントで記録される。これを常時追従（follow）して検知。
- 連続発火によるスパムを防ぐため**スロットリング**（既定 **10 分**に 1 回）。
- スロットルは**種別ごとに独立**。全滅通知が直前の切替通知に抑制されないようにするため。

> 用語: **常駐サービス（daemon）** … バックグラウンドで動き続けるプログラム。ここでは systemd user service として登録し、サーバ起動時・プロセス異常終了時も自動で立ち上がる。
> 用語: **スロットリング（throttling）** … 短時間に大量発生するイベントの通知頻度を制限すること。

## 1. アーキテクチャ

```mermaid
flowchart TD
    GW["OpenClaw Gateway"] -->|フォールバック判断を記録| LOG["gateway ログ<br/>(model_fallback_decision)"]
    SVC["openclaw-fallback-notify.service<br/>(systemd user)"] -->|openclaw logs --follow --json| LOG
    SVC --> D1{"finalOutcome ?"}
    D1 -->|next_fallback| T1{"切替: 10分<br/>スロットル"}
    D1 -->|chain_exhausted<br/>(fallbackConfigured:true)| T2{"全滅: 10分<br/>スロットル"}
    T1 -->|送信| M1["⚠️ 切替通知"]
    T2 -->|送信| M2["🛑 全滅通知"]
    M1 --> DISCORD["Discord (NEXUS チャット)"]
    M2 --> DISCORD
```

## 2. 検知するログイベント

gateway ログ（`openclaw logs --json`）の `model_fallback_decision` イベント行を評価し、`fallbackStepFinalOutcome` で分岐する。

```json
{"event":"model_fallback_decision", ...,
 "fallbackStepFromModel":"anthropic/claude-opus-4-8",
 "fallbackStepToModel":"google/gemini-2.5-flash",
 "fallbackStepFinalOutcome":"next_fallback",   // or "chain_exhausted"
 "nextCandidateProvider":"google",
 "nextCandidateModel":"gemini-2.5-flash",
 "fallbackConfigured":true}
```

- **切替**: `fallbackStepFinalOutcome":"next_fallback"` → `nextCandidateProvider`/`nextCandidateModel`（切替先）を通知。
- **全滅**: `fallbackStepFinalOutcome":"chain_exhausted"` **かつ** `fallbackConfigured":true`（fallback を設定済みで全候補を試し尽くした）→ 全滅通知。
- `fallbackConfigured":false` の `chain_exhausted`（そもそも fallback 未設定）は対象外。

## 3. 構成ファイル

### 3.1 通知スクリプト

`~/.openclaw/scripts/fallback-notify.sh`（実行権限付与: `chmod +x`）

要点:
- `openclaw logs --follow --json --no-color` でログを追従し、`while read` で 1 行ずつ評価。
- `model_fallback_decision` を含む行のみ処理し、`fallbackStepFinalOutcome` で「切替」/「全滅」に分岐。
- `notify <kind> <msg>` が**種別別**のスロットル（`~/.openclaw/state/fallback-notify.<kind>.last` に最終通知の epoch を保存）を見て、`FALLBACK_NOTIFY_THROTTLE_SEC`（既定 600 秒）未満なら抑制。`kind` は `fallback` / `exhausted`。
- 送信: `openclaw message send --channel discord --target user:<DISCORD_USER_ID> -m "..."`。
- ログ追従プロセスが落ちても systemd が再起動するため、スクリプト自体は単純な追従ループに徹する。

通知メッセージ:
- 切替: `⚠️ [フォールバック発生] Claude(primary) が応答不可のため <model> に切り替えました（HH:MM JST）。会話は継続できています。…`
- 全滅: `🛑 [全モデル失敗] Claude(primary) も fallback(Gemini 等) も全て応答不可です（HH:MM JST）。自動復旧できていません。手動確認を推奨します。…`

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
Environment=FALLBACK_NOTIFY_THROTTLE_SEC=600
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
| サービス稼働 | `systemctl --user status` | OK active (running)、throttle=600 |
| 切替検知 | 実ログの `next_fallback` 行を判定処理に通す | OK `google/gemini-2.5-flash` を抽出し「切替」分岐 |
| 全滅検知 | `chain_exhausted` + `fallbackConfigured:true` の合成行 | OK 「全滅」分岐 |
| 誤検知防止 | 旧 `chain_exhausted` + `fallbackConfigured:false` 行 | OK 通知せず（正しく無視） |
| 通知到達 | `openclaw message send` でテスト送信 | OK Discord 受信 |

> 参考: 2026-06-07 12:21(UTC) に実際のフォールバックが 1 度発火していた（"Claude CLI failed" → `google/gemini-2.5-flash`）。

## 6. 運用・調整

- **スロットル間隔の変更**: サービスの `FALLBACK_NOTIFY_THROTTLE_SEC` を編集 → `systemctl --user daemon-reload && systemctl --user restart openclaw-fallback-notify.service`。
- **停止 / 無効化**: `systemctl --user disable --now openclaw-fallback-notify.service`。
- **ログ確認**: `journalctl --user -u openclaw-fallback-notify.service -f`。
- **状態リセット**: `rm ~/.openclaw/state/fallback-notify.*.last`（次回イベントで即通知）。

## 7. 既知の制約・今後

- gateway 再起動の瞬間に発生したイベントは、追従プロセス再接続までの極短時間で取りこぼす可能性がある（systemd 再起動は数秒）。
- スロットル中（10 分以内）の同種イベントは通知されない。切替と全滅は別カウントなので、片方の抑制が他方に影響することはない。

## 8. 関連

- [[010_DONE_SETUP_gemini-fallback]] — フォールバック本体の構成
- ログ CLI: <https://docs.openclaw.ai/cli/logs>
- メッセージ送信 CLI: `openclaw message send`
