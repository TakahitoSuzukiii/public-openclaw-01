# AL2023 セキュリティアドバイザリ 週次サマリ（2026-07-25）

作成日: 2026-07-25 / STATUS: INFO / TOPIC: ALAS / 対象: Critical+Important

Amazon Linux 2023（AL2023）の公式セキュリティアドバイザリ（ALAS＝Amazon Linux Advisory System、AWS がパッケージの脆弱性と修正を告知する仕組み）を週次で監視した結果です。前回（2026-07-18）からの差分を掲載します。

## 概要（新規アドバイザリの重大度別件数）

- **Critical（緊急）**: 0 件
- **Important（重要）**: 85 件
- **Medium（中）**: 16 件
- **Low（低）**: 1 件

本記事は対象である **Critical + Important** を扱います。今回 Important は 85 件と多数のため、詳細取得したのは先頭 30 件です（本文末尾の注記参照）。内容としては大きく **NVIDIA ドライバ関連**・**Linux カーネル本体**・**カーネル livepatch（稼働中カーネルへの無停止パッチ）** の 3 系統に集約されます。

---

## Important（重要）— 主要トピック

### 1. NVIDIA Display Driver の境界外書き込み（CVE-2026-24193）

対象パッケージ（同一 CVE でまとめて公開）:
`nvlink5` / `nvidia-xconfig` / `nvidia-persistenced` / `nvidia-open` / `nvidia-modprobe` / `nvidia-settings` / `nvidia-kmod-common` / `nvidia-imex` / `nvidia-fabricmanager` / `libnvsdm` / `libnvidia-nscq` / `nvidia-driver` / `kmod-nvidia-open-dkms` / `kmod-nvidia-latest-dkms` / `cuda-drivers` / `cuda-compat`

- ALAS ID: [ALAS2023NVIDIA-2026-298](https://alas.aws.amazon.com/AL2023/ALAS2023NVIDIA-2026-298.html) 〜 [ALAS2023NVIDIA-2026-313](https://alas.aws.amazon.com/AL2023/ALAS2023NVIDIA-2026-313.html)（計 16 件）
- 重大度: Important
- 関連 CVE: CVE-2026-24193

**要約:** NVIDIA の Display Driver（Windows / Linux 版）に、境界外書き込み（out-of-bounds write＝確保したメモリ領域の外に書き込んでしまう不具合）が存在します。攻撃が成立すると、サービス妨害（DoS）、権限昇格、情報漏えい、データ改ざん、任意コード実行に至る可能性があります。GPU を使うインスタンス（EC2 の G/P 系など）でこれらパッケージを導入している場合が対象です。

### 2. Linux カーネル本体（kernel）

- ALAS ID: [ALAS2023-2026-2001](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-2001.html)
- 重大度: Important
- 対象パッケージ: kernel
- 関連 CVE: CVE-2026-45859 / CVE-2026-46324 / CVE-2026-53078 / CVE-2026-53359

**要約:** カーネルの複数の脆弱性がまとめて修正されました。内訳は、netfilter の nfnetlink_queue におけるセグメンテーション前チェックの不備（CVE-2026-45859）、nf_tables の netlink フック解放処理の不具合（CVE-2026-46324）、BPF の sock_ops における同一レジスタ参照での境界外読み取り／ポインタ漏えい（CVE-2026-53078）、KVM（x86）でのシャドウページング解放後利用（use-after-free、CVE-2026-53359）です。ネットワークフィルタや仮想化（KVM）を使う環境では影響範囲が広く、優先度の高い更新です。

### 3. NVIDIA Container Toolkit の競合状態（CVE-2026-24260）

- ALAS ID: [ALAS2023NVIDIA-2026-297](https://alas.aws.amazon.com/AL2023/ALAS2023NVIDIA-2026-297.html)
- 重大度: Important
- 対象パッケージ: nvidia-container-toolkit
- 関連 CVE: CVE-2026-24260

**要約:** NVIDIA Container Toolkit（コンテナから GPU を使うためのツール群）に TOCTOU（time-of-check time-of-use＝検査した時点と実際に使う時点の間で状態が変わる競合）による脆弱性があります。成立するとコード実行・権限昇格・データ改ざんの恐れがあります。GPU コンテナ（Docker / containerd 経由）を運用している場合は要注意です。

### 4. カーネル livepatch（CVE-2026-53362）

- ALAS ID: [ALAS2023LIVEPATCH-2026-216](https://alas.aws.amazon.com/AL2023/ALAS2023LIVEPATCH-2026-216.html) 〜 [ALAS2023LIVEPATCH-2026-227](https://alas.aws.amazon.com/AL2023/ALAS2023LIVEPATCH-2026-227.html)（計 12 件）
- 重大度: Important
- 対象パッケージ: kernel-livepatch（対象カーネルバージョンごとに個別配布: 6.1.x / 6.12.x / 6.18.x 系）
- 関連 CVE: CVE-2026-53362

**要約:** IPv6 のページ割り当て経路で fraggap（フラグメント間の隙間）を正しく計上していなかった不具合が修正されました（livepatch＝サーバを再起動せず稼働中カーネルに当てるパッチ）。対象カーネルバージョンを使っている場合、再起動なしで適用できるのが利点です。バージョンごとに別アドバイザリとして 12 件出ています。

---

## Medium / Low（新規・件数のみ）

Medium（16 件）は主に開発言語・ミドルウェア系: `php8.2`〜`php8.5`（各系列）、`tomcat9` / `tomcat10`、`jackson-databind`、`golang`（および `golang-*` 関連ツール群）、`kernel6.12`、`krb5`、`credentials-fetcher` など。Low（1 件）は `python-pygments`（[ALAS2023-2026-1973](https://alas.aws.amazon.com/AL2023/ALAS2023-2026-1973.html)）。詳細は各 ALAS ページを参照してください。

---

## 対処方法（参考・実行は管理者）

AL2023 の公式手順です。いずれも **sudo（管理者権限）が必要**なため、本監視タスクでは実行しません。適用は鈴木さん（管理者）の判断でお願いします。

```bash
# リリース更新の有無を確認
sudo dnf check-release-update

# 更新可能パッケージの確認
sudo dnf check-update

# セキュリティ更新のみ適用
sudo dnf upgrade --security
```

- 出典: [AL2023 パッケージとオペレーティングシステムの更新の管理（AWS 公式）](https://docs.aws.amazon.com/ja_jp/linux/al2023/ug/managing-repos-os-updates.html)
- NVIDIA / GPU 系は該当インスタンスにドライバを入れている場合のみ対象です。livepatch は対象カーネルバージョンでのみ意味を持ちます。

> **注記:** 今回の Critical/Important は 85 件あり、本記事は詳細取得した先頭 30 件（NVIDIA ドライバ・カーネル・livepatch に集約）を掲載しています。残りも同系統（主に NVIDIA / livepatch のバージョン別）の可能性が高いですが、網羅性が必要な場合は [ALAS 公式一覧](https://alas.aws.amazon.com/alas2023.html) を確認してください。

---

## Author and Ownership / 著作権と所属について

This project was created as a personal initiative and is not connected to any organization or group.
It is published as an individual creative work.

本プロジェクトは個人の活動として作成したものであり、特定の組織や団体の業務とは関係ありません。
個人の創作物として公開しています。
