# openclaw 週次自動アップデートタスク — 構築手順

- ステータス: DONE / SETUP
- 実施日: 2026-06-11
- 関連: タスクボード #17
- 目的: openclaw を毎週自動でアップデートする仕組みを構築する。

---

## 0. 結論（要約）

- openclaw は **gateway サービスのプロセス内（＝OpenClaw cron の agentTurn 内）からは update 不可**。
  - 二段ガード: ①env `OPENCLAW_SERVICE_MARKER=openclaw` 判定、②gateway PID が祖先プロセスかの判定。
- そのため #16（claude）と違い OpenClaw cron 方式は使えず、**gateway の外で動く systemd user タイマー**で実装。
- 構成: **systemd user timer/service（毎週金 06:20 JST）が update を実行** → 更新があった時のみ gateway 再起動 → **別の OpenClaw cron（毎週金 06:30 JST）が結果を Discord 通知**。
- 即時実行テスト成功: 現在最新（2026.6.5）のため `up-to-date`・再起動なしを確認。更新検出時の再起動経路も実機確認済み。
- sudo 不要（すべて user 権限。`systemctl --user`、Linger=yes 済み）。

---

## 1. 前提・調査（なぜ systemd 方式か）

`openclaw update` を gateway 配下のシェル／cron agentTurn から実行すると拒否される:

```
openclaw update detected it is running inside the gateway process tree.
Gateway PID <pid> is an ancestor of this process, so this updater cannot
safely stop or restart the gateway that owns it.
```

- 判定実装（dist/update-cli）: `OPENCLAW_SERVICE_MARKER?.trim() !== "openclaw"` で gateway 外と判定。加えて gateway PID の祖先判定。
- → gateway の子孫でなく、当該 env を持たない **独立した systemd user unit** から実行すれば回避可能（検証済み）。
- 権限: `~/.npm-global`（npm global）配下の更新。**sudo 不要**。`systemctl --user` 稼働・`Linger=yes`（未ログインでもタイマー発火）。

## 2. 構成要素

| ファイル | 役割 |
|---|---|
| `~/.openclaw/scripts/openclaw-autoupdate.sh` | 更新実行スクリプト（gateway 外前提）。結果を JSON 出力 |
| `~/.config/systemd/user/openclaw-autoupdate.service` | oneshot サービス。スクリプトを実行（gateway マーカー env を持たない） |
| `~/.config/systemd/user/openclaw-autoupdate.timer` | 毎週金 06:20 JST に service を起動 |
| OpenClaw cron `weekly-openclaw-autoupdate-notify` | 毎週金 06:30 JST。結果 JSON を読んで Discord 通知 |
| `~/.openclaw/state/openclaw-autoupdate-last.json` | 直近結果（ts/before/after/status/rc） |

### スクリプトの動作
1. `openclaw --version`（before）
2. `openclaw update --yes --no-restart`（env から gateway マーカーを除去して実行＝二重防御。まず再起動せず更新のみ適用）
3. `openclaw --version`（after）
4. 判定: rc≠0→`failed` / before≠after→`updated`（**この時だけ** `systemctl --user restart openclaw-gateway.service`）/ 同一→`up-to-date`（再起動しない）
5. 結果 JSON を state ファイルへ書き出し（バージョン文字列のみ。update 詳細出力は機密混入回避のため保存しない）

> 設計意図: 「更新があった週だけ gateway を再起動」。最新の週は無駄な再起動をしない。

### systemd timer（タイムゾーン）
- ホストは UTC 運用。systemd 252+ は OnCalendar に IANA タイムゾーン指定可。
- `OnCalendar=Fri *-*-* 06:20:00 Asia/Tokyo`（= Thu 21:20 UTC）。`Persistent=true`。

### 通知の分離（なぜ別 cron か）
- update は gateway 外で実行する必要があり、Discord 通知は OpenClaw（gateway 内）経由でしか送れない（外部 curl での provider 送信は禁止）。
- → 役割分担: **systemd が更新**、**OpenClaw cron が通知**。06:20 更新／06:30 通知で、更新時の gateway 再起動（実測 約2〜3分）後に通知が走るよう余裕を確保。

## 3. セットアップ手順（実行コマンド）

```bash
# 1. スクリプト配置（実行権限付与）
chmod +x ~/.openclaw/scripts/openclaw-autoupdate.sh

# 2. systemd unit 配置後
systemctl --user daemon-reload
systemd-analyze calendar "Fri *-*-* 06:20:00 Asia/Tokyo"   # 次回時刻の確認
systemctl --user enable --now openclaw-autoupdate.timer
systemctl --user list-timers openclaw-autoupdate.timer --all

# 3. 通知用 OpenClaw cron は cron ツールで登録（毎週金 06:30 JST, agentTurn/isolated）

# 4. 即時実行テスト（gateway 外で実行される）
systemctl --user start openclaw-autoupdate.service
cat ~/.openclaw/state/openclaw-autoupdate-last.json
journalctl --user -u openclaw-autoupdate.service -n 20 --no-pager
```

## 4. 即時実行テスト結果

| 項目 | 結果 |
|---|---|
| gateway 内からの `openclaw update` | ❌ 想定どおり拒否（祖先 PID 判定） |
| systemd service 経由の update（最新時） | ✅ `status=up-to-date` rc=0 |
| up-to-date 時の gateway 再起動 | ✅ 改良版で「再起動なし」を確認（pid 不変） |
| 更新検出時の再起動経路 | ✅ 初回テスト時に gateway 再起動→復帰を実機確認 |
| timer 次回発火 | ✅ 毎週金 06:20 JST |
| 通知 cron | ✅ 毎週金 06:30 JST 登録 |

> 補足: 初回の素のスクリプト（`openclaw update --yes`）は最新でも gateway を再起動したため、`--no-restart`＋更新時のみ再起動へ改良した。

## 5. 運用・ロールバック

```bash
# 一時停止 / 解除
systemctl --user disable --now openclaw-autoupdate.timer
systemctl --user enable  --now openclaw-autoupdate.timer

# 手動実行（gateway 外）
systemctl --user start openclaw-autoupdate.service

# 完全撤去
systemctl --user disable --now openclaw-autoupdate.timer
rm ~/.config/systemd/user/openclaw-autoupdate.{service,timer}
systemctl --user daemon-reload
# 通知 cron は OpenClaw cron ツールで remove
```

- 失敗時: service 失敗は journald に記録。通知 cron が `failed` を Discord 通知。failureAlert も発報。
- 手動更新はいつでも「gateway 外のシェル」から `openclaw update`。

## 6. #16（claude）との違い（まとめ）

| | claude（#16） | openclaw（#17） |
|---|---|---|
| 更新方式 | symlink 切替（native installer） | npm dist ツリー置換 |
| gateway 内から更新 | 可（無停止） | **不可**（祖先 PID 判定で拒否） |
| スケジューラ | OpenClaw cron（agentTurn が直接 `claude update`） | **systemd user timer**（gateway 外）＋通知用 OpenClaw cron |
| gateway 再起動 | 不要 | 更新があった時のみ |
