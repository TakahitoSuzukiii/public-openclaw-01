# claude CLI（Claude Code）週次自動アップデートタスク — 構築手順

- ステータス: DONE / SETUP
- 実施日: 2026-06-11
- 関連: タスクボード #16
- 目的: claude CLI（Claude Code）を毎週自動でアップデートする OpenClaw cron ジョブを構築する。

---

## 0. 結論（要約）

- 毎週 **木曜 06:20 JST** に `claude update` を自動実行する OpenClaw cron ジョブを登録。
- claude は **native installer 版**（`~/.local/share/claude/versions/<ver>` 配下 ＋ `~/.local/bin/claude` シンボリックリンク切替）。更新は**実行中セッションに影響せず、gateway 再起動も不要・sudo 不要**。
- 即時実行テスト成功: **2.1.170 → 2.1.173** に更新。

---

## 1. 前提・調査

- インストール形態: `~/.local/bin/claude` → `~/.local/share/claude/versions/<version>` への symlink。
- 更新コマンド: `claude update`（別名 `claude upgrade`）= 組み込みの self-update。最新版を取得して symlink を差し替える方式。
- 権限: ユーザー領域（`~/.local`）のみ。**sudo 不要**。
- 無停止性: 更新は新バージョンを別ディレクトリへ展開して symlink を張り替えるだけなので、稼働中の openclaw gateway や実行中の claude プロセスには影響しない（次回起動分から新版）。

> 公式参考: Claude Code のアップデート機構（`claude update`）。

## 2. 即時実行テスト（初回・手動）

```bash
claude --version            # [before] 2.1.170 (Claude Code)
claude update < /dev/null   # Successfully updated from 2.1.170 to version 2.1.173
claude --version            # [after]  2.1.173 (Claude Code)
```

- 結果: ✅ 2.1.170 → 2.1.173 に更新成功。セッション断なし。
- `< /dev/null` は `claude` 実行時の「no stdin data received」警告を抑止するため付与。

## 3. cron ジョブ登録（OpenClaw cron）

既存の週次タスク（github-trending 等）と同じ構成（`isolated` セッション ＋ `agentTurn` ＋ `cron`/Asia/Tokyo ＋ Discord announce ＋ failureAlert）。

| 項目 | 値 |
|---|---|
| name | `weekly-claude-cli-autoupdate` |
| schedule | cron `20 6 * * 4`（毎週木曜 06:20）、tz `Asia/Tokyo` |
| sessionTarget | `isolated` |
| payload.kind | `agentTurn` |
| model | `anthropic/claude-opus-4-8` |
| timeoutSeconds | 600 |
| delivery | announce → Discord（owner DM） |
| failureAlert | after=1 → Discord |

ジョブの処理内容（agentTurn メッセージ要旨）:
1. `claude --version`（旧版）
2. `claude update < /dev/null`（更新・出力捕捉）
3. `claude --version`（新版）
4. 旧版→新版を判定し、結果を1行で Discord 通知（例: `✅ claude CLI 自動更新: 2.1.170 → 2.1.173（更新あり）` / `✅ 既に最新` / `⚠️ 更新失敗: <原因>`）

セキュリティ: 機密（トークン/パスワード/秘密鍵/個人情報）は出力・ログ・通知に残さない。sudo 不使用。

## 4. 検証結果

| 項目 | 結果 |
|---|---|
| 更新コマンドの動作（即時テスト） | ✅ 2.1.170 → 2.1.173 |
| cron ジョブ登録 | ✅ 登録済（次回: 翌木曜 06:20 JST） |
| gateway 再起動の要否 | 不要（symlink 切替方式） |
| sudo の要否 | 不要 |

## 5. 運用メモ・ロールバック

- ジョブ無効化: OpenClaw cron の `update`（`enabled:false`）または `remove`。
- 手動更新: いつでも `claude update` でOK。
- 失敗時は failureAlert が Discord に通知（連続失敗1回で発報）。

## 6. 注意（cron の自動失効について）

- 本ジョブは OpenClaw Gateway の永続 cron（Claude Code セッション内 CronCreate の7日失効とは別物）。Gateway 稼働中は継続実行される。
