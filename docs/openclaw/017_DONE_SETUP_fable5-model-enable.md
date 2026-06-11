# Fable 5（claude-fable-5）を OpenClaw で使えるようにする — セットアップ手順

- ステータス: DONE / SETUP
- 実施日: 2026-06-11
- 関連: タスクボード #47（Fable 調査）／調査ドキュメント `docs/info/research/20260611_INFO_RESEARCH_claude-fable5-vs-opus48.md`
- 目的: Claude Fable 5 を本サーバでモデル指定（オーバーライド）して使える状態にする。**デフォルトモデルは変更しない**（`anthropic/claude-opus-4-8` のまま）。

---

## 0. 結論（要約）

- Fable 5 はこのアカウントの **claude-cli OAuth 経由で利用可能**（ライブ疎通テスト成功）。
- OpenClaw の config にモデルを明示登録済み。**gateway 再起動は不要**だった（稼働中の gateway がそのままルーティング受理）。
- 既定モデルは `opus-4-8` のまま。Fable はセッション単位のモデルオーバーライドで使う想定。

---

## 1. 前提・調査

- 本サーバのモデルランタイムは `claude-cli`（OAuth、認証プロファイル `anthropic:claude-cli`）。
- `agents.defaults.models` に各モデルを明示登録し、それぞれ `agentRuntime.id = "claude-cli"` を割り当てる構成。
- 未登録モデルはビルトインランタイム（APIキー必須）にフォールバックして失敗しうるため、Fable も明示登録が必要。
- claude CLI（v2.1.170）は `fable` をエイリアスとしてネイティブ対応（`--model fable`）。

## 2. 実施手順

### 手順1: ランタイムが Fable を解釈できるか確認（読み取り）

```bash
claude --help | grep -A2 -- "--model"   # 'fable' がエイリアス例に含まれることを確認
```

### 手順2: ライブ疎通テスト（最小トークンの実呼び出し）

```bash
claude --model fable -p "Reply with exactly: FABLE_OK and your model id"
# → 期待出力: FABLE_OK claude-fable-5
```

成功 = アカウントに Fable 利用権限あり・OAuth 経由で到達可能。

### 手順3: config へモデル登録（バックアップ → 追記）

対象: `~/.openclaw/openclaw.json` の `agents.defaults.models`

```bash
cp -a ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.pre-fable   # バックアップ
```

`anthropic/claude-opus-4-8` の直後に追記（既存は全保持・追記のみ）:

```json
"anthropic/claude-fable-5": {
  "agentRuntime": {
    "id": "claude-cli"
  }
}
```

> 注意: `agents.defaults.model.primary` は `anthropic/claude-opus-4-8` のまま変更しない。

### 手順4: 構文検証

```bash
node -e "JSON.parse(require('fs').readFileSync(process.env.HOME+'/.openclaw/openclaw.json','utf8')); console.log('JSON valid')"
```

### 手順5: 反映確認（再起動不要）

セッションのモデルオーバーライドで Fable を選択 → 認証解決を確認 → デフォルトへ戻す。

- Fable 選択時の status カードに `Session selected: anthropic/claude-fable-5 · oauth (anthropic:claude-cli)` が出れば OK。
- 稼働中 gateway がそのまま受理したため、**gateway restart は実施せず**。
- ※ config 登録は将来の新規セッション／gateway 再起動後も claude-cli ルーティングを保証する目的で必須。

## 3. 使い方（運用）

- 既定は `opus-4-8`。Fable を使いたいセッションだけモデルを `anthropic/claude-fable-5` に指定する。
- セッション内: `/model anthropic/claude-fable-5`（戻す: `/model anthropic/claude-opus-4-8` または `/reset`）。
- コスト注意: Fable は Opus 4.8 の約2倍（入力 $10 / 出力 $50 per MTok）。常用せず「ここぞ」の場面のみ。

## 4. API 仕様の注意（Fable 固有・claude-api 経由で直接叩く場合）

- `temperature`／`top_p`／`top_k` は渡すと 400（サンプリングはプロンプトで制御）。
- 思考を使う: `thinking: {type:"adaptive"}`。使わない: `thinking` パラメータ自体を**書かない**（`disabled` を渡すと 400。Opus 4.8 と挙動が異なる点）。
- アシスタント・プレフィルは 400。大きい出力（〜128K）はストリーミング必須。
- 詳細は調査ドキュメント `docs/info/research/20260611_INFO_RESEARCH_claude-fable5-vs-opus48.md` を参照。

## 5. ロールバック

```bash
cp -a ~/.openclaw/openclaw.json.pre-fable ~/.openclaw/openclaw.json
```

（登録を消すだけ。デフォルトモデルは元から変更していないため運用影響なし。）

## 6. 検証結果

| 項目 | 結果 |
|---|---|
| claude CLI が `fable` を解釈 | ✅ |
| ライブ疎通（OAuth 実呼び出し） | ✅ `FABLE_OK claude-fable-5` |
| config 登録・JSON 妥当性 | ✅ |
| OpenClaw 側オーバーライド受理 | ✅ `oauth (anthropic:claude-cli)` で解決 |
| デフォルトモデル維持 | ✅ `anthropic/claude-opus-4-8` |
