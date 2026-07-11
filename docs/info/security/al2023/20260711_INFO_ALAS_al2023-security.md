# AL2023 セキュリティアドバイザリ 週次まとめ（2026-07-11）

作成日: 2026-07-11 / STATUS: INFO / TOPIC: ALAS / 対象: Critical+Important

Amazon Linux 2023（AL2023）向けに AWS が公開した公式セキュリティアドバイザリ（ALAS: Amazon Linux Security Advisory＝Amazon Linux のセキュリティ勧告）のうち、今週新規に検知した分をまとめます。情報源は AWS 公式 ALAS フィードのみです。

## 概要（新規アドバイザリの重大度別件数）

- **Critical（緊急）**: 0 件
- **Important（重要）**: 27 件 ← 本記事の対象
- **Medium（中）**: 10 件
- **Low（低）**: 2 件

今週は Critical はありませんが、Important が 27 件と多めです。特に Linux カーネルのローカル権限昇格（LPE）、libheif の RCE 可能性、Firefox の多数の脆弱性が目立ちます。用語補足: RCE（Remote Code Execution＝遠隔コード実行）、LPE（Local Privilege Escalation＝ローカル権限昇格）、DoS（Denial of Service＝サービス妨害）、OOB（Out-Of-Bounds＝境界外アクセス）、UAF（Use-After-Free＝解放済みメモリ参照）。

---

## Important（重要）アドバイザリ 一覧

### カーネル系（優先度高: ローカル権限昇格を含む）

#### [ALAS2023-2026-1936](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1936.html) — kernel
- 重大度: Important / 対象パッケージ: `kernel`（修正版へ更新）
- 関連CVE: （個別CVE番号は未付与のフィード。net フラグメント処理の修正）
- 要約: IPv4/IPv6 のページ割り当てフラグメント処理（`__ip_append_data()` / `__ip6_append_data()`）で fraggap の計算ミスにより 8〜15 バイトのヒープバッファオーバーフローが発生します。**非特権ローカルユーザ（ユーザ名前空間・コンテナ内を含む）が権限昇格・任意コード実行・コンテナ脱出・DoS・情報漏えいに悪用できる**、影響の大きい問題です。カーネル系の中でも最優先で適用を検討してください。

#### [ALAS2023-2026-1937](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1937.html) — kernel6.12
- 重大度: Important / 対象パッケージ: `kernel6.12`
- 要約: 1936 と同一の fraggap 会計ミスによるヒープオーバーフロー。kernel6.12 系列向けの修正です。悪用時の影響は 1936 と同じく権限昇格・コンテナ脱出等。

#### [ALAS2023-2026-1938](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1938.html) — kernel6.18
- 重大度: Important / 対象パッケージ: `kernel6.18`
- 要約: 1936 と同一の fraggap 会計ミスによるヒープオーバーフロー。kernel6.18 系列向けの修正です。影響は同上。

#### [ALAS2023-2026-1924](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1924.html) — kernel
- 重大度: Important / 対象パッケージ: `kernel`
- 関連CVE: CVE-2026-43303, CVE-2026-43492, CVE-2026-45850 ほか多数（計約65件）
- 要約: Linux カーネルの多数の脆弱性をまとめて修正する大規模ロールアップです。mm/page_alloc の解放処理、crypto の整数アンダーフロー、ipvs/udf/netfilter など広範なサブシステムのメモリ安全性・DoS 修正を含みます。CVE 件数が非常に多いため、通常のカーネル更新として適用を推奨します。

#### [ALAS2023-2026-1925](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1925.html) — kernel6.18
- 重大度: Important / 対象パッケージ: `kernel6.18`
- 関連CVE: CVE-2026-46054
- 要約: SELinux の overlayfs 上での `mmap()` / `mprotect()` のアクセスチェック不備を修正します。アクセス制御が正しく適用されないケースの是正です。

### ランタイム / コンテナ系

#### [ALAS2023-2026-1920](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1920.html) — nodejs24
- 重大度: Important / 対象パッケージ: `nodejs24`
- 関連CVE: CVE-2026-11525, CVE-2026-11822, CVE-2026-11824, CVE-2026-12151 ほか計20件
- 要約: Node.js 24 系のセキュリティ更新。同梱の undici が Set-Cookie の SameSite 属性を部分一致で誤解釈する問題（`SameSite=NoneOfYourBusiness` を None 扱いにする等）や、SQLite（FTS5）由来のメモリ破損など多数を修正します。サーバ応答の Cookie を扱うアプリで影響が出やすい点に注意。

#### [ALAS2023-2026-1921](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1921.html) — nodejs22
- 重大度: Important / 対象パッケージ: `nodejs22`
- 関連CVE: CVE-2026-11525, CVE-2026-11822 ほか計17件
- 要約: Node.js 22 系向けの同種修正。undici の SameSite 誤解釈および SQLite 関連のメモリ破損などを含みます。

#### [ALAS2023-2026-1901](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1901.html) — docker
- 重大度: Important / 対象パッケージ: `docker`
- 関連CVE: CVE-2026-27145, CVE-2026-41567, CVE-2026-42504, CVE-2026-42507
- 要約: Docker/Moby の脆弱性修正。Go 標準の x509 ホスト名検証が大量の DNS SAN で二次的に計算量が増える（DoS 要因）問題や、Moby 側の複数の問題を含みます。コンテナ基盤として広く使われるため優先度は高めです。

#### [ALAS2023-2026-1902](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1902.html) — ecs-init
- 重大度: Important / 対象パッケージ: `ecs-init`
- 関連CVE: CVE-2026-25680, CVE-2026-25681, CVE-2026-27136, CVE-2026-27145 ほか計9件
- 要約: Amazon ECS のエージェント初期化パッケージ。Go の HTML パーサでの過剰 CPU 消費（DoS）や、サニタイズ後でも意図しない HTML ツリーになり XSS を招く問題、x509 検証の計算量問題などを修正します。

### メディア / 画像・動画コーデック系（RCE・OOB に注意）

#### [ALAS2023-2026-1919](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1919.html) — libheif
- 重大度: Important / 対象パッケージ: `libheif`
- 関連CVE: CVE-2026-47178, CVE-2026-49271
- 要約: HEIF/AVIF 画像処理ライブラリで、クロマサブサンプリング（4:2:0 等）のタイル画像を復号する際にオフセットのスケーリング漏れがあり、境界外書き込みが発生します。**遠隔コード実行（RCE）につながりうる**重大な問題で、画像を開くだけで悪用される恐れがあります。

#### [ALAS2023-2026-1918](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1918.html) — libde265
- 重大度: Important / 対象パッケージ: `libde265`
- 関連CVE: CVE-2026-49295, CVE-2026-49337, CVE-2026-49346
- 要約: H.265（HEVC）デコーダで、参照ピクチャセットの結合数チェック漏れにより 16 要素配列を超える境界外書き込みが起きます。細工した H.265 ストリームで悪用可能です。

#### [ALAS2023-2026-1917](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1917.html) — gstreamer1-plugins-bad-free
- 重大度: Important / 対象パッケージ: `gstreamer1-plugins-bad-free`
- 関連CVE: CVE-2026-52718 〜 CVE-2026-52722, CVE-2026-53702
- 要約: GStreamer の AV1 パーサでバイト数とビット数を取り違え、パーサ同期ずれ・アサート異常でクラッシュ（DoS）します。VA JPEG デコーダの境界外読み取りも含みます。細工メディアを開かせる攻撃が想定されます。

#### [ALAS2023-2026-1926](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1926.html) — gstreamer1-plugins-good
- 重大度: Important / 対象パッケージ: `gstreamer1-plugins-good`
- 関連CVE: CVE-2026-39043, CVE-2026-39044, CVE-2026-53705
- 要約: Matroska(MKV) demuxer の bz2 圧縮トラックでバッファサイズ計算の括弧漏れによりヒープオーバーフロー、WAV パーサの cue チャンクで整数オーバーフローが発生します。細工した動画ファイルで悪用され得ます。

#### [ALAS2023-2026-1923](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1923.html) — rclone
- 重大度: Important / 対象パッケージ: `rclone`
- 関連CVE: CVE-2026-46601, CVE-2026-46602, CVE-2026-46604
- 要約: rclone 同梱の Go 画像ライブラリで、WebP デコーダの panic、TIFF タイルサイズ無制限によるメモリ枯渇、境界外ストリップオフセットでの panic を修正します。主に DoS 系です。

### Web / ネットワークサーバ系

#### [ALAS2023-2026-1922](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1922.html) — nginx
- 重大度: Important / 対象パッケージ: `nginx`
- 関連CVE: CVE-2026-42055, CVE-2026-48142
- 要約: HTTP/2 プロキシ（`proxy_http_version 2` / `grpc_pass`）利用時、`ignore_invalid_headers off` かつ `large_client_header_buffers` が 2MB 超という特定条件で、巨大ヘッダによりワーカープロセスにヒープオーバーフローが起き再起動（DoS）します。該当構成の場合は要注意です。

#### [ALAS2023-2026-1915](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1915.html) — haproxy
- 重大度: Important / 対象パッケージ: `haproxy`
- 関連CVE: CVE-2026-55204
- 要約: HAProxy の HPACK 動的テーブル挿入（`hpack_dht_insert()`）で、メモリプール枯渇時に `hpack_dht_defrag()` の戻り値検証が不足し NULL ポインタ参照でワーカーがクラッシュ（DoS）します。

#### [ALAS2023-2026-1899](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1899.html) — samba
- 重大度: Important / 対象パッケージ: `samba`
- 関連CVE: CVE-2026-3012
- 要約: 証明書の自動登録 GPO が有効なドメインメンバーで、CA 証明書が平文 HTTP で取得され信頼ストアに導入されるため、中間者攻撃で任意の証明書を差し込まれる恐れがあります。該当 GPO 設定を使う環境で影響します。

### ライブラリ / パーサ系

#### [ALAS2023-2026-1900](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1900.html) — expat
- 重大度: Important / 対象パッケージ: `expat`
- 関連CVE: CVE-2026-56132
- 要約: libexpat 2.8.2 未満で、複数パーサ間のデータ構造共有時に scaffold 配列の再確保処理が不適切となり `doProlog` でヒープバッファオーバーフローが発生します。

#### [ALAS2023-2026-1913](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1913.html) — expat
- 重大度: Important / 対象パッケージ: `expat`
- 関連CVE: CVE-2026-56403, CVE-2026-56406, CVE-2026-56407
- 要約: libexpat 2.8.2 未満で `storeAtts`・`XML_ParseBuffer`・`doProlog` の各所に整数オーバーフローがあります。XML 解析処理を持つアプリで影響します（上記 1900 と併せて expat の更新を推奨）。

#### [ALAS2023-2026-1903](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1903.html) — sqlite
- 重大度: Important / 対象パッケージ: `sqlite`
- 関連CVE: CVE-2026-11822, CVE-2026-11824
- 要約: SQLite 3.53.2 未満の FTS5 全文検索拡張にメモリ破損があり、細工した DB に対して MATCH クエリを実行すると境界外読み取り・ヒープオーバーフロー書き込みが発生します。クラッシュ・メモリ枯渇・任意コード実行の恐れ。

#### [ALAS2023-2026-1916](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1916.html) — util-linux
- 重大度: Important / 対象パッケージ: `util-linux`
- 関連CVE: CVE-2026-53615
- 要約: libblkid の DOS パーティション解析（`libblkid/src/partitions/dos.c`）で整数オーバーフロー/ラップアラウンドが発生します。

### 言語ランタイム（Python 各系列）

#### [ALAS2023-2026-1910](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1910.html) — python3.13
- 重大度: Important / 関連CVE: CVE-2026-11940, CVE-2026-1502, CVE-2026-3276, CVE-2026-7210, CVE-2026-7774, CVE-2026-8328, CVE-2026-9669, CVE-2025-4330, CVE-2021-4189
- 要約: Python 3.13 のセキュリティ更新。`tarfile.extractall()` の data/tar フィルタがハードリンク細工で回避される問題ほか、複数の脆弱性を修正します。

#### [ALAS2023-2026-1929](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1929.html) — python3.14
- 重大度: Important / 関連CVE: CVE-2026-11940, CVE-2026-3276, CVE-2026-7774, CVE-2026-8328, CVE-2025-4330, CVE-2021-4189
- 要約: Python 3.14 向けの同種修正（tarfile フィルタ回避ほか）。

#### [ALAS2023-2026-1930](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1930.html) — python3.11
- 重大度: Important / 関連CVE: CVE-2025-13462, CVE-2026-11940, CVE-2026-7210, CVE-2026-8328, CVE-2025-4330, CVE-2021-4189
- 要約: Python 3.11 向け。tarfile が AREGTYPE ブロックを DIRTYPE に誤正規化し、複数実装間で tar の解釈が食い違う問題（CVE-2025-13462）ほかを修正。

#### [ALAS2023-2026-1931](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1931.html) — python3.12
- 重大度: Important / 関連CVE: CVE-2025-13462, CVE-2026-11940, CVE-2026-7210, CVE-2026-8328, CVE-2025-4330, CVE-2021-4189
- 要約: Python 3.12 向けの同種修正（tarfile 誤正規化・フィルタ回避ほか）。

### アプリケーション / ツール系

#### [ALAS2023-2026-1928](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1928.html) — firefox
- 重大度: Important / 対象パッケージ: `firefox`
- 関連CVE: CVE-2026-12289 ほか計42件
- 要約: Firefox の大規模なセキュリティ更新。WebRender の権限昇格、HTTP コンポーネントの解放済みメモリ参照（UAF）、多数のメモリ安全性バグを Firefox 152 / ESR 140.12 / 115.37 系で修正します。デスクトップで Firefox を使う場合は更新推奨。

#### [ALAS2023-2026-1932](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1932.html) — ansible
- 重大度: Important / 対象パッケージ: `ansible`
- 関連CVE: CVE-2026-11837
- 要約: `ansible.posix` の authorized_key モジュールが `os.chown()`（`lchown` でなく）を使い O_NOFOLLOW なしでファイルを開くため、非特権ユーザが `~/.ssh` にシンボリックリンクを仕込むと、root 実行時に任意パスの所有権を奪えるローカル権限昇格が可能です。

---

## medium / low の新規アドバイザリ（参考・詳細は割愛）

- **Medium（10 件）**: opensc(1898), wireshark(1904), aws-nitro-tpm-tools(1905), nerdctl(1908), containerd(1909), libxml2(1911), ImageMagick(1914), python3.9(1933), kernel6.12(1934), kernel(1935)
- **Low（2 件）**: gnupg2(1912), cifs-utils(1927)

---

## 対処方法（参考・実行はシステム管理者）

以下は AL2023 公式のパッケージ更新手順です。**いずれも `sudo`（管理者権限）が必要なため、本監視タスクでは実行しません。** 適用は管理者の判断で行ってください。

```bash
# リリース更新の有無を確認
sudo dnf check-release-update

# 更新可能パッケージを確認
sudo dnf check-update

# セキュリティ更新のみを適用
sudo dnf upgrade --security
```

出典（AL2023 公式ドキュメント）: https://docs.aws.amazon.com/ja_jp/linux/al2023/ug/managing-repos-os-updates.html

> 注記: 本記事は今回の Critical/Important（計 27 件、Critical 0 / Important 27）をすべて掲載しています（reportableTruncated=false）。修正バージョンの具体値は各 ALAS ページおよび `dnf` の出力を参照してください。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
