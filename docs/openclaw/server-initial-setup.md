# OpenClaw サーバ初期セットアップ手順書

OpenClaw を稼働させるための Linux サーバ初期構築手順をまとめたものです。
Amazon Linux 2023（AL2023）on EC2 を想定しています。

- **作成日:** 2026-05-31
- **対象 OS:** Amazon Linux 2023（AL2023）/ x86_64
- **想定スペック:** メモリ 2GB 以上（小さい場合は Swap 追加で補う）
- **作業ユーザ:** 初期 `ec2-user` → 作業用 `<your-user>` へ切り替え
- **前提:** AWS EC2 にて起動済み、SSH で `ec2-user` 接続可能

---

## 📚 用語ミニ辞典

| 略語 | 綴り | 意味 |
|---|---|---|
| `SSH` | Secure Shell | 暗号化された遠隔ログインプロトコル |
| `sudo` | superuser do | 一時的に管理者権限でコマンドを実行する仕組み |
| `wheel` | — | sudo を使える管理者グループ（Red Hat 系の慣習名） |
| `dnf` | Dandified YUM | AL2023 標準のパッケージマネージャ |
| `Swap` | — | メモリが足りないとき HDD/SSD を仮想メモリとして使う仕組み |
| `npm` | Node Package Manager | Node.js のパッケージ管理ツール |
| `OAuth` | Open Authorization | パスワードを渡さず認可するプロトコル |

---

## 🗂 全体の流れ

1. 管理ユーザ作成（`<your-user>`）
2. SSH 鍵セットアップ
3. SSH ハードニング（`ec2-user` ログイン禁止）
4. システム更新
5. 基本パッケージ確認・インストール（git / node / npm）
6. Node.js を 24.x にアップグレード
7. Swap 設定（2GB）
8. Claude Code インストール
9. OpenClaw インストール
10. 動作確認

---

## 1. 管理ユーザ作成

### 🎯 目的
EC2 デフォルトの `ec2-user` の代わりに、作業用ユーザ `<your-user>` を作成する。

### 🔑 必要権限
root（`sudo` または `ec2-user` で sudo 可）

### 💻 コマンド

```bash
# ユーザを作成（-m でホームディレクトリも作成）
sudo useradd -m <your-user>

# sudo 可能グループ（wheel）に追加
sudo usermod -aG wheel <your-user>

# 作成確認
id <your-user>

# パスワード設定（対話）
sudo passwd <your-user>
```

### 📖 解説
- `useradd -m` の `-m` は「ホームディレクトリ `/home/<your-user>` を同時に作る」オプション
- `wheel` グループ所属で `sudo` が使えるようになる（AL2023 既定）
- パスワードは強固なものを設定し、`<元ファイルパス>` を参照する形でドキュメントには残さない

### ✅ 確認方法
```bash
id <your-user>
# uid=1001(<your-user>) gid=1001(<your-user>) groups=1001(<your-user>),10(wheel) のように表示される
```

---

## 2. SSH 鍵セットアップ

### 🎯 目的
パスワード認証ではなく、SSH 公開鍵で `<your-user>` にログインできるようにする。

### 🔑 必要権限
root（`/home/<your-user>/` 配下の所有権付与のため）

### 💻 コマンド

```bash
# .ssh ディレクトリ作成（700 = 本人のみ rwx）
sudo mkdir /home/<your-user>/.ssh
sudo chmod 700 /home/<your-user>/.ssh
sudo chown <your-user>:<your-user> /home/<your-user>/.ssh

# 公開鍵を貼り付け（vi で編集 → 1行に公開鍵を貼って保存）
sudo vi /home/<your-user>/.ssh/authorized_keys

# 権限を厳格化（600 = 本人のみ rw）
sudo chmod 600 /home/<your-user>/.ssh/authorized_keys
sudo chown <your-user>:<your-user> /home/<your-user>/.ssh/authorized_keys
```

### 📖 解説
- `authorized_keys` には**公開鍵**を1行ずつ記入する（秘密鍵は絶対に置かない）
- パーミッションが緩いと SSH 側が安全のためにキーを無視する。`700` / `600` は必須
- 公開鍵は手元の `~/.ssh/id_ed25519.pub` などの内容をコピーして貼る

### ✅ 確認方法
別ターミナルから `ssh <your-user>@<server-ip>` で鍵ログインできること。
**この時点ではまだ `ec2-user` でのログインも生きている。失敗したら復旧用にこのセッションは残しておく。**

---

## 3. SSH ハードニング（`ec2-user` ログイン禁止）

### 🎯 目的
不要な共通ユーザ `ec2-user` でのログインを禁止し、攻撃面を減らす。

### 🔑 必要権限
root

### 💻 コマンド

```bash
# 現状確認
sudo cat /etc/ssh/sshd_config

# バックアップ（.org として保存）
sudo cp -prf /etc/ssh/sshd_config /etc/ssh/sshd_config.org

# 編集
sudo vi /etc/ssh/sshd_config
```

エディタで**ファイル最下部**に以下を追記：

```
DenyUsers ec2-user
```

```bash
# 反映前確認 → 再起動 → 反映後確認
sudo systemctl status sshd
sudo systemctl restart sshd
sudo systemctl status sshd
```

### 📖 解説
- `DenyUsers` は指定ユーザの SSH ログインを拒否する（ローカル `su` は可能）
- バックアップ `.org` を残しておくと、設定ミス時に `sudo cp .org` で戻せる
- 反映前に**別ターミナルで `<your-user>` ログインができることを必ず確認**してから ec2-user を禁止する

### ✅ 確認方法
- `<your-user>` でログインできる
- `ec2-user` でログインしようとすると拒否される（`Permission denied`）

---

## 4. システム更新

### 🎯 目的
最新のセキュリティパッチ・パッケージ更新を当てる。

### 🔑 必要権限
root

### 💻 コマンド

```bash
# 状態確認（任意）
sudo dnf check-release-update
sudo dnf check-update
sudo dnf check-upgrade

# 更新適用
sudo dnf update -y
```

### 📖 解説
- `check-release-update` は AL のバージョンアップ確認
- `check-update` は通常のパッケージ更新確認（差分一覧）
- `update -y` で全件更新を確認なしで実行

### ✅ 確認方法
```bash
sudo dnf check-update
# 「No packages marked for update.」と出れば最新
```

---

## 5. 基本パッケージ確認・インストール

### 🎯 目的
git / node / npm がインストールされているか確認し、無ければインストール。

### 🔑 必要権限
root

### 💻 コマンド

```bash
# 環境確認
sudo cat /etc/os-release
sudo uname -m
sudo git --version
sudo node --version
sudo npm --version

# インストール（既に入っていればスキップされる）
sudo dnf install -y git
sudo dnf install -y nodejs
sudo dnf install -y npm
```

### 📖 解説
- AL2023 標準リポジトリの Node.js は古め（次のステップで 24.x に更新する）
- `--version` が `command not found` を返したらインストール必要

### ✅ 確認方法
すべて `--version` が値を返すこと。

---

## 6. Node.js を 24.x にアップグレード

### 🎯 目的
OpenClaw が要求する Node.js 24.x（LTS 系の最新メジャー）に揃える。

### 🔑 必要権限
root（リポジトリ追加・パッケージインストール）

### 💻 コマンド

```bash
# 現状の node を確認（複数バージョン混在の有無）
which -a node

# NodeSource の Node.js 24.x リポジトリを追加
curl -fsSL https://rpm.nodesource.com/setup_24.x | sudo bash -

# 24.x をインストール（既存より優先される）
sudo dnf install -y nodejs

# バージョン確認
node --version       # 例: v24.x.x
node -v
npm prefix -g        # グローバル npm のインストール先（PATH 反映用）
echo "$PATH"

# グローバル npm の bin を PATH に追加（このシェルセッションのみ）
export PATH="$(npm prefix -g)/bin:$PATH"
```

### 📖 解説
- `NodeSource` は Node.js の公式系サードパーティ RPM リポジトリ
- `npm prefix -g` でグローバルインストール先を取得（例: `/usr/local`）
- 永続化したい場合は `~/.bashrc` に `export PATH="..."` を追記する

### ✅ 確認方法
```bash
node --version
# v24.x.x が表示されること
```

### 💡 永続化（推奨）
```bash
echo 'export PATH="$(npm prefix -g)/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

---

## 7. Swap 設定（2GB）

### 🎯 目的
小スペック EC2（t3.micro 等）でメモリ不足によるプロセス停止を防ぐ。

### 🔑 必要権限
root

### 💻 コマンド

```bash
# 現状確認
swapon --show
free -h
cat /proc/swaps
ls -lh /swapfile         # 存在しなければエラーが出る
grep swap /etc/fstab
cat /proc/sys/vm/swappiness

# 2GB の swap ファイルを作成
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# 永続化（再起動後も自動マウント）
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

### 📖 解説
- `fallocate -l 2G` で 2GB のスパースファイルを高速作成
- `mkswap` で swap 領域としてフォーマット
- `swapon` で有効化
- `/etc/fstab` に追記しないと再起動で消える

### 💡 swappiness の推奨値
`vm.swappiness` は「メモリが余っていてもどれだけ swap を使うか」のしきい値（0〜100）。

| 値 | 動作 | 用途 |
|---|---|---|
| 60（既定） | 比較的早く swap を使う | 一般デスクトップ |
| **10〜30**（推奨） | 物理メモリを優先、swap は最後の砦 | サーバ用途 |
| 0 | OOM 寸前まで swap を使わない | DB サーバ等、I/O 重視 |

設定例（永続化）:
```bash
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.d/99-swappiness.conf
sudo sysctl --system
cat /proc/sys/vm/swappiness    # 10 になっていればOK
```

### ✅ 確認方法
```bash
swapon --show
# NAME      TYPE SIZE USED PRIO
# /swapfile file   2G   0B   -2
free -h
# Swap: の行が 2.0Gi になっている
```

---

## 8. Claude Code インストール

### 🎯 目的
Anthropic 公式 CLI ツール「Claude Code」をインストールする。

### 🔑 必要権限
- `/opt/claude` 作成は root
- 以降の `claude` コマンドは一般ユーザで OK

### 💻 コマンド

```bash
# 設置ディレクトリ作成
sudo mkdir -p /opt/claude
cd /opt/claude

# 公式インストールスクリプト
# 公式ドキュメント: https://code.claude.com/docs/ja/quickstart
curl -fsSL https://claude.ai/install.sh | bash

# 動作確認
claude --version

# OAuth 認証（ブラウザが開く → ログイン → トークン取得）
claude auth login

# 認証状態確認
claude auth status

# 自己更新
claude update

# 環境診断
claude doctor

# バックグラウンドデーモン状態確認
claude daemon status
```

### 📖 解説
- `claude auth login` はブラウザを開いて Anthropic アカウントで OAuth 認証する
- `claude doctor` は環境チェック（PATH、依存、API 接続性など）
- `claude daemon` は補完候補キャッシュなどを担う常駐プロセス

### 💡 `claude daemon status` 出力例
```
Status: not running
Mode: ephemeral
```
- `not running` でも `ephemeral`（必要時のみ起動）モードなら正常
- 常駐させたい場合は `claude daemon start` で起動可

### ✅ 確認方法
```bash
claude doctor
# Diagnostics, Updates, Background server などのセクションが ✓ で並ぶ
```

---

## 9. OpenClaw インストール

### 🎯 目的
OpenClaw（Claude Code 等を統合管理するゲートウェイ・エージェント基盤）をインストールする。

### 🔑 必要権限
- `/opt/openclaw` 作成は root
- 以降の `openclaw` コマンドは一般ユーザで OK

### 💻 コマンド

```bash
# 設置ディレクトリ作成
sudo mkdir -p /opt/openclaw
cd /opt/openclaw

# 公式インストールスクリプト
curl -fsSL https://openclaw.ai/install.sh | bash

# 動作確認
openclaw --version
openclaw doctor
openclaw gateway status

# 自己更新
openclaw update

# Anthropic 認証（OAuth、ブラウザ起動）
openclaw models auth login --provider anthropic

# 認証済みアカウント一覧
openclaw models auth list

# 再度診断
openclaw doctor

# モデル接続性確認
openclaw models status --provider anthropic
```

### 📖 解説
- `openclaw models auth login --provider anthropic` は Claude Code とは別に OpenClaw 用に OAuth トークンを取得
  - Claude Code の OAuth と共通化したい場合は `openclaw configure` で auth-profile を再利用設定可能
- `openclaw gateway status` で WebSocket ゲートウェイ（既定 `127.0.0.1:18789`）の起動を確認
- `openclaw doctor` は OpenClaw 環境総合診断（Memory search、Discord 設定、auth 期限など）

### 💡 補足：Claude Code 認証の再利用
Claude Code 側で `claude auth login` 済みであれば、OpenClaw の `claude-cli` プロファイルとして再利用される。`openclaw models auth list` に `anthropic:claude-cli` が出ていれば OK。

### ✅ 確認方法
```bash
openclaw doctor
# Memory search / Skills / Plugins / Model auth 等が ✓ で並ぶ
```

---

## 10. 動作確認チェックリスト

セットアップ完了時点で以下が成立していることを確認：

- [ ] `<your-user>` で SSH 鍵ログインができる
- [ ] `ec2-user` で SSH ログインできない（拒否される）
- [ ] `sudo dnf check-update` で「最新」と出る
- [ ] `node --version` が `v24.x.x`
- [ ] `swapon --show` に 2GB swap が表示される
- [ ] `claude doctor` がエラーなし
- [ ] `openclaw doctor` がエラーなし
- [ ] `openclaw gateway status` が `running`

---

## 🔒 セキュリティチェック（任意・推奨）

```bash
# OpenClaw 設定の機密情報スキャン
openclaw secrets audit --check

# 詳細セキュリティ監査
openclaw security audit --deep
```

**重要:** トークン・APIキーなどの実値は **`.env` または secret manager** に置き、本ドキュメントを含むあらゆる場所で `<REDACTED>` 表記を徹底する（SOUL.md セキュリティ原則）。

---

## 📎 関連

- Anthropic Claude Code 公式: <https://code.claude.com/docs/ja/quickstart>
- OpenClaw 公式: <https://openclaw.ai/>
- AL2023 リリースノート: <https://docs.aws.amazon.com/linux/al2023/release-notes/>
