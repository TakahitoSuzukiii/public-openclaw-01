# WSL 常駐化（ログオン不要）＋ ショートカットログイン（Win11 / Hermes・Optimus）

> ステータス: DONE / カテゴリ: SETUP / 実施日 2026-08-09
> ⚠️ マスキング規約準拠。ユーザー名/ホスト名は placeholder（`<your-user>` / `<hostname>`）。Windows パスワードは一切記載しない。
> 目的: Windows の電源を入れれば（**ログインしなくても**）Hermes/Optimus が自動で常駐し、`optimus` コマンドで一発ログインできるようにする。

## 0. 全体像

Hermes gateway は WSL 内の **user systemd サービス＋linger** で動くが、**WSL 自体は Windows 起動時に自動起動しない**（誰かが WSL を呼ぶまで停止）。そこで：

1. **電源設定**でスリープ/休止を無効（24/7）
2. **Task Scheduler**（起動時・ログオン不要）で WSL を常駐起動（`sleep infinity` で生かし続ける）
3. **PowerShell プロファイル関数**で `optimus` ログインコマンドを用意

起動連鎖: 電源ON → タスク発火（ログオン不要）→ `wsl … sleep infinity` → systemd(PID1) → linger が hermes-gateway 起動 → Discord 接続 → Optimus 応答可能。

## 1. 電源設定（24/7）

設定 → システム → 電源:
- **スリープ = なし**、**休止状態 = なし**（電源接続時）
- 画面オフ（例 1時間）は可（ディスプレイを消すだけで本体/WSL は稼働継続）
- ノート PC は「バッテリー駆動」側は別設定 → 常時稼働は AC 接続前提

## 2. WSL 常駐化（Task Scheduler・ログオン不要）

事前確認（干渉チェック）:
```powershell
Get-ScheduledTask -TaskName "KeepWSLHermes" -ErrorAction SilentlyContinue
Test-Path "$env:USERPROFILE\keep-wsl.vbs"
Get-ScheduledTask | ? { $_.TaskName -match 'wsl|hermes|ubuntu' } | Select TaskName,State
```

設定（管理者 PowerShell）:
```powershell
# (1) 隠れ起動用 VBS（ウィンドウを出さずに WSL を常駐起動）
$vbs = 'CreateObject("WScript.Shell").Run "wsl -d Ubuntu -u <your-user> -e sh -c ""exec sleep infinity""", 0, False'
Set-Content -Path "$env:USERPROFILE\keep-wsl.vbs" -Value $vbs -Encoding ASCII

# (2) 起動時タスク（ログオン不要＝run whether logged on or not）
$vbsPath  = "$env:USERPROFILE\keep-wsl.vbs"
$action   = New-ScheduledTaskAction -Execute "wscript.exe" -Argument "`"$vbsPath`""
$trigger  = New-ScheduledTaskTrigger -AtStartup
$settings = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries -StartWhenAvailable -ExecutionTimeLimit ([TimeSpan]::Zero)
$pw = Read-Host "Windows ログインパスワードを入力"   # 資格情報として Windows が保存（本ドキュメントには残さない）
Register-ScheduledTask -TaskName "KeepWSLHermes" -Action $action -Trigger $trigger -Settings $settings -User "$env:COMPUTERNAME\$env:USERNAME" -Password $pw -RunLevel Limited

# (3) 動作テスト
Start-ScheduledTask -TaskName "KeepWSLHermes"; Start-Sleep 5; wsl -l -v   # Ubuntu Running
```

- `LogonType = Password`（ログオン不要）／`AtStartup` トリガーで登録される。
- **PIN ログイン運用でも、素の Windows パスワードが必要**（`account.microsoft.com` 等で確認/リセット）。パスワード保存を避けたい場合は `-AtLogOn` 版（ログイン後に発火・パスワード不要）で代替可。

## 3. 挙動と制約

**耐性（止まらない条件）**
- 端末に入って `exit` → 維持（`sleep infinity`）
- ログアウト → 維持（linger）
- 再起動 / Windows Update → 自動復帰（AtStartup）
- **ログインしていなくても起動**（run whether logged on or not）

**制約（正直な前提）**
- **PC の電源が入っていること**（クラウドサーバと違い個人 PC）。
- **スリープ/休止で一時停止**（復帰時に再開）→ 24/7 は電源設定で無効化。
- **再起動直後は 30〜60 秒ほど無反応**（WSL 起動→systemd→gateway→MCP 接続リトライ→Discord 接続 の初期化。特に未認証の MCP のリトライ分）。1 分ほど待てば応答。

## 4. ショートカットログイン（`optimus`）

PowerShell プロファイルに関数を追加:
```powershell
if (!(Test-Path $PROFILE)) { New-Item -ItemType File -Path $PROFILE -Force }
Add-Content $PROFILE 'function optimus { wsl -d Ubuntu --cd "~" }'
```

プロファイルが `AllSigned`/`Restricted` で読めない場合は、CurrentUser を `RemoteSigned` に:
```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
. $PROFILE
```
- `RemoteSigned`（CurrentUser 限定）＝ローカル作成スクリプト（プロファイル）は実行可／DL スクリプトは署名必須。既存への影響は軽微。
- 以後 `optimus` で WSL に入り、Linux ホーム `~` に着地。

**代替（実行ポリシーを変えたくない場合・.bat 方式）**:
```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\bin" | Out-Null
'@echo off'              | Set-Content "$env:USERPROFILE\bin\optimus.bat"
'wsl -d Ubuntu --cd "~"' | Add-Content "$env:USERPROFILE\bin\optimus.bat"
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";$env:USERPROFILE\bin", "User")
```
→ cmd / PowerShell / Win+R から `optimus`。

## 5. 可逆性（元に戻す）

```powershell
Unregister-ScheduledTask -TaskName "KeepWSLHermes" -Confirm:$false
Remove-Item "$env:USERPROFILE\keep-wsl.vbs" -ErrorAction SilentlyContinue
# optimus 関数はプロファイルから該当行を削除
```

## 6. 検証（本番テスト）

1. `optimus` で WSL に入れる（`pwd` が `/home/<your-user>`）
2. WSL に入って `exit` → `wsl -l -v` が Running のまま
3. **Windows 再起動 → ログインせず → 1 分待って Discord で Optimus 応答**（= ログイン不要常駐の最終確認）
