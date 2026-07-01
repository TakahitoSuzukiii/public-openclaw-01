# 042_DONE_SETUP_ami-restore-20260701.md - AMI からのサーバ復元 総合記録

> STATUS: DONE / CATEGORY: SETUP / 作成日: 2026-07-01
> 旧サーバがネットワーク設定変更で SSH 不能となり終了 → 約1か月前の AMI から復元機を起動し、
> タスクボード常駐・cron・自動更新タイマー・ドキュメント整合・Tailscale を段階的に復旧した記録。
> **再発防止と再現性のため、原因・手順・検証・技術的知見（特に cron の toolsAllow 問題）を残す。**

---

## 0. 結論（要約）

- 旧サーバは「デモ公開作業中に `/etc/resolv.conf` を変更 → ネットワークインターフェース断 → SSH 不能・EIP 付与不可」で終了。
- 約1か月前の **AMI から復元機を起動**。ユーザ領域（workspace・エージェント状態・個人データ）は別バックアップから手動復元済みだったが、**システム側（systemd ユニット・cron・タイマー・Tailscale・`/opt/docs`）が AMI 時点のまま欠落**していた。
- 本作業でシステム側を順次復旧。ネットワーク系（Tailscale）は**最後に・最大限慎重に**実施し、**デモ公開は行わない**方針を厳守。
- 復旧順序: ①タスクボード常駐 → ②cron 6件 → ③GitHub PAT 更新 → ④自動更新タイマー → ⑤docs 整合 → ⑥Tailscale。各フェーズ前に AMI スナップショットを取得。

---

## 1. 経緯と原因（旧サーバ終了）

- デモサイト公開の過程で `/etc/resolv.conf`（DNS 設定）を直接変更し、HTTPS 公開を試みた。
- 結果、ネットワークインターフェースが down し **SSH 接続不能**、Elastic IP も付与できず、復旧不能となりサーバを終了。
- 教訓: **ネットワーク／DNS 設定（`/etc/hosts`・`/etc/resolv.conf` 等）の変更は事故時に自身を締め出す（lock-out）リスクが高い**。以後、これらの変更は「変更前に必ず確認」「現行 SSH セッションを維持したまま作業」を原則とする。

---

## 2. 復元の全体像（AMI とユーザ復元の差分）

| レイヤ | AMI 復元直後の状態 | 対応 |
|---|---|---|
| workspace／エージェント状態／個人データ | 別バックアップから手動復元済み（最新に近い） | 追加対応なし |
| タスクボード常駐サービス（systemd --user） | 未登録（コード・DB はあるが未起動） | §3 で復元 |
| OpenClaw cron ジョブ | 全消失 | §4 で復元 |
| 自動更新 systemd タイマー | 未登録 | §6 で復元 |
| `/opt/docs`（ドキュメントマスター） | 旧構成のまま（大幅に古い） | §7 で整合 |
| Tailscale | 未インストール | §8 で復元 |

---

## 3. タスクボード常駐サービスの復元

- アプリ本体（`server.mjs` ほか、`~/.openclaw/workspace/tasks/task-board/`）とデータ（`data/tasks.db`）は残存。**「常時起動しておく systemd 登録」だけが欠落**していた。
- 構築手順ドキュメント（`014_DONE_SETUP_task-board.md`）記載の定義どおりに `~/.config/systemd/user/openclaw-taskboard.service` を再作成し、`enable --now`。
- 待受は **`127.0.0.1:18790` の loopback 限定**（外部非公開）。
- 検証: 主要ページ・API が HTTP 200、プライベート IP からは接続拒否（loopback 限定を実測）、既存の gateway／Discord に影響なし、DB 行数が起動前後で不変（非破壊）。

---

## 4. OpenClaw cron 6件の復元（＋重要な技術的知見）

### 4.1 復元対象
バックアップ（`~/.openclaw/workspace/.backups/weekly-crons-*.json`）から 6 件を byte-exact 復元:
`weekly-al2023-security`（日06:00）/ `weekly-claude-automation-knowledge`（月06:40）/ `weekly-aws-automation-knowledge`（火06:50）/ `weekly-claude-cli-autoupdate`（木06:20）/ `weekly-openclaw-autoupdate-notify`（金06:30）/ `weekly-github-trending`（土06:30）。すべて JST。

### 4.2 ⚠️ 重要な技術的知見: cron の `toolsAllow` と claude-cli ランタイム
- cron の agentTurn は **claude-cli ランタイム**で実行される。
- **claude-cli は `toolsAllow`（ツール制限）を持つジョブを実行できない**。実行時エラー: `CLI backend claude-cli cannot enforce runtime toolsAllow; use an embedded runtime for restricted tool policy`。
- 元の 6 ジョブは `toolsAllow` を**持たない**（＝全ツール許可）から正常動作していた。
- しかし cron 管理ツール（`cron add`/`update` 相当）で作成すると **`toolsAllow` が自動注入**され、`null` 指定でも消えない。→ 作成した瞬間に claude-cli で壊れる。
- **正しい復元**: cron ジョブ格納 DB（`~/.openclaw/state/openclaw.sqlite` の `cron_jobs` テーブル）で `payload_tools_allow_json = NULL` かつ `job_json.payload.toolsAllow` を削除し、**gateway 再起動で反映**。
- 副次知見: cron ランタイムは変更後に **gateway を再起動しないとメモリキャッシュが更新されない**（DB 直接更新はライブ反映されない）。
- **運用上の注意**: これらのジョブは**以後 cron 管理ツールで編集しない**（`toolsAllow` 再注入で壊れる）。編集は DB 直接＋再起動、または CLI 書込スコープ復旧後に CLI で行う。

### 4.3 GitHub 連携の 401（真因の切り分け）
- 当初 push 系ジョブが「GitHub にアクセスできない」と誤診したが、実地プローブの結果、原因は **`toolsAllow` 注入 ＋ GitHub Personal Access Token（PAT）の失効（401 Bad credentials）** の二点だった。
- ツール自体は cron から到達可能で、`toolsAllow` を除去し PAT を更新することで解決（§5）。

---

## 5. GitHub PAT（Personal Access Token）の更新

- 復元機の PAT が失効（`GET /user` → HTTP 401 Bad credentials）。
- 新しい **fine-grained PAT**（対象リポジトリのみ・Contents 読み書き）を発行し、`~/.openclaw/.env` と `gateway.systemd.env` の該当行のみ差し替え（他の値は保持・バックアップ取得）。
- MCP サーバ（GitHub 連携）は**起動時に env を読む**ため、反映には gateway 再起動が 1 回必要。
- 検証: 更新後に認証成功（`get_me` がログインユーザを返す）を確認。
- 🔒 トークン値はドキュメント・ログ・コミットに一切残さない（マスキング規約）。

---

## 6. 自動更新 systemd タイマーの復元

- 構成（`019_DONE_SETUP_openclaw-autoupdate.md` 準拠）を再構築:
  - `~/.openclaw/scripts/openclaw-autoupdate.sh`（gateway 外で `openclaw update --yes --no-restart`、結果を JSON で state ファイルへ、**再起動しない**）
  - `~/.config/systemd/user/openclaw-autoupdate.service`（oneshot）
  - `~/.config/systemd/user/openclaw-autoupdate.timer`（毎週金 06:20 JST、`Persistent=true`）
- `Linger=yes` により未ログインでも発火。`enable --now` で次回発火を確認。
- 別途 OpenClaw cron（金 06:30）が結果を通知し、更新があった時のみ「事前 Discord 通知 → gateway 再起動」を行う分離設計（既存踏襲）。

---

## 7. ドキュメント整合（`/opt/docs` 復元）

- 正本は **GitHub 公開リポジトリ（docs）**。復元機の `/opt/docs` は旧構成（大幅に古い）、ローカルチェックアウトも一部欠落していた。
- GitHub 正本（docs ツリー）を基準に:
  - ローカルチェックアウト（`workspace/docs`）を正本で完全化（不足分を補完、内容競合 0 件）。
  - `/opt/docs` マスターを再構築（`docs/openclaw` → `/opt/docs/openclaw`、`docs/info` → `/opt/docs/openclaw-news/info`）。旧ファイルは `_trash/` へ退避（物理削除しない）。
  - ローカル限定ドキュメント（private リポジトリ バックアップ手順など）はバックアップリポから復元し、マスターへ配置。
- アプリ照合: タスクボードのコード（`server.mjs`／`src`／`public`／`deploy` ほか）は private バックアップリポと**完全一致＝不整合なし**を確認。

---

## 8. Tailscale 復旧（開発・個人利用のみ／デモ公開なし）

### 8.1 方針（前回事故の反映）
- **開発・個人利用向けのみ**復旧。**デモ公開（Funnel）は行わない**。
- 🔒 安全原則:
  - 現行 SSH セッションを維持したまま作業。
  - **`--accept-dns=false`** で `/etc/resolv.conf` を触らせない（前回事故と同じ「resolv.conf 書き換え」クラスを回避）。この機は systemd-resolved 稼働のため元々上書きされにくいが、明示指定で二重防御。
  - **`--exit-node` を使わない**（デフォルト経路を奪わない＝SSH 経路保全）。
  - **`serve`（tailnet 内・非公開）のみ**。**`funnel`（インターネット公開）は使わない**。

### 8.2 手順（インストール〜デバイス登録は sudo／ブラウザ認証）
公式: https://tailscale.com/download/linux ／ DNS 挙動: https://tailscale.com/docs/reference/faq/dns-resolv-conf ／ Serve: https://tailscale.com/kb/1312/serve
```bash
# 1) インストール（sudo）
sudo dnf config-manager --add-repo https://pkgs.tailscale.com/stable/amazon-linux/2023/tailscale.repo
sudo dnf install -y tailscale
sudo systemctl enable --now tailscaled

# 2) tailnet 参加＝デバイス登録（sudo＋ブラウザ認証）
sudo tailscale up --accept-dns=false --hostname=openclaw-server

# 3)（任意）自分の端末から taskboard へ：tailnet 内限定の Serve（非公開）
sudo tailscale serve --bg 18790
tailscale serve status
```
- クライアント側（PC／スマホ）は同一アカウントで Tailscale にログインすると相互到達可能。

### 8.3 検証結果（すべて安全）
- `tailscaled` active。tailnet に自機＋クライアント端末が参加。
- **`/etc/resolv.conf` 不変**（systemd-resolved の symlink・VPC DNS のまま）、`CorpDNS=false`（DNS 非管理）、`RouteAll=false`・ExitNode なし（デフォルト経路保全）。
- Serve は **tailnet only**（Funnel 無効）。`https://openclaw-server.<tailnet>.ts.net/` から taskboard へ HTTP 200 到達を確認。

---

## 9. AMI スナップショット方針

- 破壊的・不可逆になりうる作業（特にネットワーク系）の**直前に AMI を取得**。
- 本復元では「素の復元直後」「主要復旧後」「Tailscale 着手直前」でスナップショットを取得（取得はユーザが手動実施）。

---

## 10. 再発防止・教訓（まとめ）

1. **ネットワーク／DNS 変更は lock-out リスク最大**。変更前確認・SSH 維持・スナップショット直前取得を徹底。
2. **cron（claude-cli ランタイム）は `toolsAllow` 非対応**。ジョブは `toolsAllow` を持たせない（DB で `NULL`）。管理ツールは自動注入するため、これらのジョブは DB 直接管理。
3. **cron 変更は gateway 再起動で反映**（メモリキャッシュ）。
4. **MCP 連携の env（PAT 等）変更は gateway 再起動で反映**。PAT 失効時の 401 に注意。
5. **ドキュメント正本は GitHub**。`/opt/docs` はミラーとして再構築可能。
6. **秘密情報（トークン・パスワード・鍵）は一切ドキュメント化しない**。固有情報（ホスト名・IP・ユーザ名・tailnet 名等）は placeholder 化。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
