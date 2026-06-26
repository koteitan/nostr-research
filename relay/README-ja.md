[<< back](../README-ja.md) | [Japanese] | [English](README-en.md)

# Nostr リレー仕様

## リレーランキング (Last Checked: 2026/06/26)

出典: [nostr.watch](https://nostr.watch/relays/software)

| 名前 | リレー数 | バージョン数 | シェア | ベース |
|------|--------|----------|-------|--------|
| hoytech/strfry | 237 | 35 | 26.9% | - |
| ~gheartsfield/nostr-rs-relay | 205 | 10 | 23.3% | - |
| bitvora/haven | 71 | 15 | 8.1% | khatru |
| cameri/nostream | 55 | 8 | 6.3% | - |
| bitvora/wot-relay | 25 | 5 | 2.8% | khatru |
| fiatjaf/khatru | 18 | 3 | 2.0% | - |

## limit パラメータの動作

このドキュメントでは、異なるNostrリレー実装における `limit` フィルターパラメータの動作を比較します。

### 概要テーブル

| リレー実装 | デフォルト最大limit | 設定パラメータ | 動作 |
|-----------|-------------------|----------------|------|
| [strfry](evidences/strfry.md) | [500](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L111) | `relay.maxFilterLimit` | フィルターごとに min(client_limit, 500) 件のイベントを返す |
| [nostream](evidences/nostream.md) | [5000](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L221) | `limits.client.subscription.maxLimit` | min(client_limit, 5000) 件のイベントを返す |
| [nostr-rs-relay](evidences/nostr-rs-relay.md) | 明示的な制限なし | デフォルト設定に見つからず | limit未指定時は全てのマッチするイベントを返す |
| [khatru](evidences/khatru.md) (フレームワーク) | デフォルト制限なし | 該当なし | 実装依存、組み込みの最大limitなし |
| [haven](evidences/haven.md) (khatruベース) | [制限なし](https://github.com/bitvora/haven/blob/8d26f9e/init.go#L164) (khatru継承) | 未設定 | eventstoreを通じて全てのマッチするイベントを返す |
| [wot-relay](evidences/wot-relay.md) (khatruベース) | [500](https://github.com/bitvora/wot-relay/blob/7c5803f/main.go#L164) | `UseEventstore` 第2引数 (maxQueryLimit) | khatru の maxQueryLimit (=500) を結果サイズの上限として強制 |

## レート制限

このテーブルは、各リレーがクライアントリクエストに対して適用するレート制限を示します。

| リレー | タイプ | 最大サブスクリプション数 | イベント送信レート | フィルター/REQレート | 接続レート | 備考 |
|-------|------|----------------------|-------------------|---------------------|------------|------|
| [strfry](evidences/strfry.md) | - | [200](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L120) | デフォルトでは未設定 | 未設定 | 未設定 | `relay.maxSubsPerConnection` で接続あたりの同時REQ数を制限。イベント送信/フィルター/接続レートはコア設定としては未設定（外部のリバースプロキシ等で対応） |
| [nostream](evidences/nostream.md) | kind 0, 3, 40, 41 | - | [6 events/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L162-L169) | - | - | メタデータ、コンタクト、チャンネル作成/更新イベント |
| [nostream](evidences/nostream.md) | kind 1, 2, 4, 42 | - | [12 events/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L170-L177) | - | - | ノート、DM、チャンネルメッセージ |
| [nostream](evidences/nostream.md) | kind 5-7, 43-49 | - | [30 events/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L178-L185) | - | - | 削除、リアクション、チャンネルイベント |
| [nostream](evidences/nostream.md) | kind 10000-19999, 30000-39999 | - | [24 events/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L186-L194) | - | - | 置換可能イベント・パラメータ化置換可能イベント |
| [nostream](evidences/nostream.md) | kind 445 | - | [60 events/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L195-L199) | - | - | Marmotグループイベント (新規追加) |
| [nostream](evidences/nostream.md) | kind 20000-29999 | - | [60 events/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L200-L205) | - | - | 一時イベント |
| [nostream](evidences/nostream.md) | 全イベント | - | [720 events/hour](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L206-L208) | - | - | 全体制限 |
| [nostream](evidences/nostream.md) | その他 | [10](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L217) | - | [240 msg/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L225-L227) | [12/sec](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L117-L118) および [48/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L119-L120) | 最大サブスクリプション数、生メッセージ(REQ含む)レート、接続レート |
| [nostr-rs-relay](evidences/nostr-rs-relay.md) | - | 制限なし | 設定可能 (デフォルト: 無制限) [messages_per_sec](https://github.com/scsibug/nostr-rs-relay/blob/b5c1f64/config.toml#L115) | 設定可能 (デフォルト: 無制限) [subscriptions_per_min](https://github.com/scsibug/nostr-rs-relay/blob/b5c1f64/config.toml#L121) | 未設定 | messages_per_sec / subscriptions_per_min は未設定または 0 で無制限 |
| [khatru](evidences/khatru.md) | - | 制限なし | [2 events/3min](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/sane_defaults.go#L12) (max 10 tokens) | [20 filters/min](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/sane_defaults.go#L17) (max 100 tokens) | [1 conn/5min](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/sane_defaults.go#L21) (max 100 tokens) | フレームワーク。[`ApplySaneDefaults`](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/sane_defaults.go#L9)経由 |
| [haven](evidences/haven.md) | Private | khatru継承 | [50 events/min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L61) (max 100 tokens) | khatru継承 | [3 conn/5min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L66) (max 9 tokens) | 認証+ホワイトリスト必須リレー |
| [haven](evidences/haven.md) | Chat | khatru継承 | [50 events/min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L72) (max 100 tokens) | khatru継承 | [3 conn/3min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L77) (max 9 tokens) | WoT内ユーザー向けチャット |
| [haven](evidences/haven.md) | Inbox | khatru継承 | [10 events/min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L83) (max 20 tokens) | khatru継承 | [3 conn/min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L88) (max 9 tokens) | 受信用リレー |
| [haven](evidences/haven.md) | Outbox | khatru継承 | [10 events/60min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L94) (max 100 tokens) | khatru継承 | [3 conn/min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L99) (max 9 tokens) | 公開メッセージ/メディア配信用 |
| [wot-relay](evidences/wot-relay.md) | - | 制限なし | [5 events/min](https://github.com/bitvora/wot-relay/blob/7c5803f/main.go#L168) (max 30 tokens) | [5 filters/min](https://github.com/bitvora/wot-relay/blob/7c5803f/main.go#L202) (max 30 tokens) | [10 conn/2min](https://github.com/bitvora/wot-relay/blob/7c5803f/main.go#L205) (max 30 tokens) | khatruベース。khatruデフォルトより厳格 |

**重要な注意事項:**
- **最大サブスクリプション数**: 接続ごとの最大同時REQサブスクリプション数。「制限なし」はリレーが上限を強制しないことを意味する（フレームワーク依存）
- **トークンバケットアルゴリズム**: ほとんどのリレーはトークンが時間とともに補充されるトークンバケットレート制限を使用
- **レート制限 ≠ 結果制限**: レート制限はリクエスト頻度を制御し、リクエストごとに返されるイベント数ではない
- **IP単位 vs 接続単位**: ほとんどの実装はIPアドレス単位で制限を適用

---

## フィルター値とメッセージサイズ制限

このテーブルは、REQフィルターで指定できる値の数（authors, ids, kinds, #tags など）とメッセージサイズの制限を示します。

### フィルター値制限

| リレー | フィルター値制限 | REQあたり最大フィルター数 | 設定パラメータ | 備考 |
|-------|----------------|------------------------|---------------|------|
| [strfry](evidences/strfry.md) | 制限なし | [200](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L99) | `relay.maxReqFilterSize` | メッセージサイズ制限のみ。`relay.maxTagsPerFilter`（デフォルト3）でフィルターあたりのタグフィルター数も制限 |
| [nostream](evidences/nostream.md) | [2500](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L219) (合計) | [10](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L218) | `limits.client.subscription.maxFilterValues` | 全フィルター値の合計。最小プレフィックス長は [4](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L222)、サブスクリプションID長は最大 [256](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L220) |
| [nostr-rs-relay](evidences/nostr-rs-relay.md) | 制限なし | 制限なし | - | メッセージサイズ制限のみ |
| [khatru](evidences/khatru.md) | 制限なし | 制限なし | - | フレームワークにデフォルト制限なし |
| [haven](evidences/haven.md) | 制限なし (khatru継承) | 制限なし (khatru継承) | - | khatru継承。Chat/Inbox/Outboxでは空フィルター・複雑フィルターを拒否 |
| [wot-relay](evidences/wot-relay.md) | 制限なし | 制限なし | - | khatru継承。NoComplexFilters によりタグ2個超かつ要素4個超のフィルターは拒否 |

### メッセージサイズ制限

| リレー | WebSocketメッセージ | イベントサイズ | コンテンツサイズ | 設定パラメータ |
|-------|-------------------|--------------|-----------------|---------------|
| [strfry](evidences/strfry.md) | [131,072バイト (128 KB)](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L96) | [65,536バイト (64 KB)](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L21) | - | `relay.maxWebsocketPayloadSize`, `events.maxEventSize` |
| [nostream](evidences/nostream.md) | [524,288バイト (512 KB)](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L77) | - | [102,400バイト (100 KB)](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L147) (kind別) | `network.maxPayloadSize`, `limits.event.content[].maxLength` |
| [nostr-rs-relay](evidences/nostr-rs-relay.md) | [131,072バイト (128 KB)](https://github.com/scsibug/nostr-rs-relay/blob/b5c1f64/config.toml#L138) | [131,072バイト (128 KB)](https://github.com/scsibug/nostr-rs-relay/blob/b5c1f64/config.toml#L135) | - | `limits.max_ws_message_bytes`, `limits.max_event_bytes` |
| [khatru](evidences/khatru.md) | [512,000バイト (500 KB)](https://github.com/fiatjaf/khatru/blob/9f99b98/relay.go#L45) | - | - | `MaxMessageSize` |
| [haven](evidences/haven.md) | [512,000バイト (500 KB)](https://github.com/fiatjaf/khatru/blob/v0.19.1/relay.go) (khatru継承) | - | - | khatru `MaxMessageSize` |
| [wot-relay](evidences/wot-relay.md) | [512,000 (500 KB)](https://github.com/bitvora/wot-relay/blob/7c5803f/main.go#L140) | - | - | `MaxMessageSize` (khatru継承) |

### authors 配列の実質的な最大数

pubkey 1件 = 64文字 (hex) + 約3文字 (引用符・カンマ) ≈ 67 bytes として計算:

| リレー | 制限要因 | 最大 authors 数 (概算) |
|-------|---------|----------------------|
| nostream | `maxFilterValues: 2500` | **2,500 人** |
| strfry | WebSocket 128 KB | ~1,900 人 |
| nostr-rs-relay | WebSocket 128 KB | ~1,900 人 |
| khatru系 | WebSocket 500 KB | ~7,400 人 |

**注意事項:**
- **nostream**: `maxFilterValues` は authors, ids, kinds, #tags など全てのフィルター値の合計
- **その他**: 明示的なフィルター値制限はなく、メッセージサイズ制限が実質的な上限となる
- フィルター値が多すぎるとリレーのパフォーマンスに影響する可能性がある

---

## 時間ベースの制限

### イベント送信時刻の検証

このテーブルは、クライアントが新しいイベントを送信する際に、リレーが `created_at` タイムスタンプをどのように検証するかを示します。

| リレー | 最大未来オフセット | 最大過去オフセット | 備考 |
|-------|------------------|------------------|------|
| [strfry](evidences/strfry.md) | [+900秒 (15分)](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L24) | [-94,608,000秒 (約3年)](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L27) | この範囲外のイベントを拒否 |
| [nostream](evidences/nostream.md) | [+900秒 (15分)](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L144) | [制限なし (0)](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L145) | `maxNegativeDelta: 0` は過去方向の制限を無効化 |
| [nostr-rs-relay](evidences/nostr-rs-relay.md) | [+1,800秒 (30分)](https://github.com/scsibug/nostr-rs-relay/blob/b5c1f64/config.toml#L105) | 制限なし | 未来のイベントのみ拒否 |
| [khatru](evidences/khatru.md) | 強制なし | 強制なし | フレームワークはデフォルトでは強制しない |
| [haven](evidences/haven.md) | 強制なし (khatru継承) | 強制なし (khatru継承) | khatruの動作を継承 |
| [wot-relay](evidences/wot-relay.md) | 強制なし | 強制なし | khatruの動作を継承 |

**時刻検証の目的:**
- **未来オフセット制限**: クライアントが遠い未来のタイムスタンプを持つイベントを作成することを防止
- **過去オフセット制限**: 悪意のあるユーザーが非常に古いタイムスタンプでイベントを作成するバックデート攻撃を防止

### イベント保存/削除ポリシー

このテーブルは、リレーが時間の経過とともに保存されたイベントをどのように管理するかを示します。

| リレー | 一時イベント経過時間 | 一時イベント生存期間 | 通常イベント最大経過時間 | 備考 |
|-------|-------------------|-------------------|----------------------|------|
| [strfry](evidences/strfry.md) | [60秒](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L30)より古い場合拒否 | [300秒](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L33)後に自動削除 | - | Kind 20000-29999のみ |
| [nostream](evidences/nostream.md) | - | - | - | 一時イベントは配送のみ。通常イベントの保存期間は `event.retention.maxDays: -1` で無期限 |
| [nostr-rs-relay](evidences/nostr-rs-relay.md) | - | - | - | 自動削除・保持期間ポリシー未実装 |
| [khatru](evidences/khatru.md) | - | - | - | 実装依存 |
| [haven](evidences/haven.md) | - | - | - | 自動削除なし |
| [wot-relay](evidences/wot-relay.md) | - | - | [365日](https://github.com/bitvora/wot-relay/blob/7c5803f/.env.example#L27)後に削除 (ただしコードのデフォルトは [0=無効](https://github.com/bitvora/wot-relay/blob/7c5803f/main.go#L283)) | MAX_AGE_DAYSで設定可能 |

**主な違い:**
- **一時イベント** (kind 20000-29999): 長期保存を想定していない一時的なイベント
- **通常イベント**: リレーが通常無期限に保存する標準イベント
- **自動削除**: 一部のリレーはストレージを管理するために古いイベントを自動的に削除

---


## 詳細分析

### 1. strfry (C++)

**設定ファイル:** `strfry.conf`

**デフォルト設定:**
```conf
relay {
    # Maximum records that can be returned per filter
    maxFilterLimit = 500
}
```

**動作:**
- クライアントが `limit: N` でイベントをリクエストした場合、strfryはフィルターごとに **min(N, 500)** 件のイベントを返す
- クライアントがlimitを指定しないか、500より大きいlimitを指定した場合、strfryは500に制限
- これはサブスクリプションごとではなく、フィルターごとの制限

**レート制限:**
- 最大サブスクリプション数（接続あたりの同時REQ数）: 200 (`relay.maxSubsPerConnection`)
- イベント送信レート/フィルターレート/接続レートはコア設定としては未設定

**サイズ制限:**
- イベントサイズ: 65,536バイト (64 KB) - 正規化JSON
- WebSocketペイロード: 131,072バイト (128 KB)
- タグ値サイズ: 1,024バイト
- 最大タグ数: 2,000
- REQごとの最大フィルター数: 200

**時間制限:**
- 未来に900秒 (15分) を超えるイベントを拒否
- 94,608,000秒 (約3年) より古いイベントを拒否
- 60秒より古い一時イベントを拒否
- 一時イベント生存期間: 300秒 (5分)

**サポートNIP:** 1, 2, 4, 9, 11, 28, 40, 45, 70, 77（NIP-45はCOUNTが有効な場合、NIP-77はnegentropyが有効な場合、NIP-42はAUTHの`serviceUrl`を設定した場合に追加される）

---

### nostream

nostreamはTypeScriptで実装されたリレーで、設定はすべて `resources/default-settings.yaml` に集約されており、7つの観点すべてがファイル単位で版管理されている。確認バージョンは `33f0ba9` (2026-06-25)。

**limitパラメータ**: デフォルトの最大limitは `limits.client.subscription.maxLimit: 5000` で、クライアントが `limit: N` を指定すると `min(N, 5000)` 件が返される。あわせて最大同時サブスクリプション数 10、サブスクリプションごとの最大フィルター数 10、合計最大フィルター値 2500 が強制される。

**レート制限**: nostreamの最大の特徴はイベントkindごとに細分化されたイベント送信レートである。kind 0/3/40/41 が6 events/min、kind 1/2/4/42 が12 events/min、kind 5-7/43-49 が30 events/min、置換可能/パラメータ化置換可能イベント(10000-19999, 30000-39999)が24 events/min、一時イベント(20000-29999)が60 events/min、Marmotグループイベント(kind 445)が60 events/min、そして全体で720 events/hourの上限が掛かる。これとは別に、生メッセージ(REQを含む)に対して240 msg/min、接続に対して12/sec・48/minのレート制限がある。レート制限戦略は指数加重移動平均(`ewma`)を採用している。

**時間ベースの制限**: `createdAt.maxPositiveDelta: 900` により未来方向には最大15分先までのイベントを許容する。`maxNegativeDelta: 0` は過去方向の制限を無効化しており、実質的に過去オフセットは制限なしとなる。一時イベント(kind 20000-29999)はDBに保存されず配送のみ行われ、通常イベントの保存期間は `event.retention.maxDays: -1` で無期限である。

**フィルター値とサイズ制限**: `maxFilterValues: 2500` がauthors・ids・kinds・#tagsなど全フィルター値の合計上限となる。したがってauthors数も実質的にこの上限に従う。サイズ面ではネットワークペイロード上限が `network.maxPayloadSize: 524288` (512 KB)、イベントコンテンツ上限が `limits.event.content[].maxLength: 102400` (100 KB) で、コンテンツ上限はkind範囲(0-10/40-49, 11-39/50-max)ごとに設定できる。

**サポートNIP**: package.jsonの `supportedNips` に基づき、NIP-01, 02, 03, 04, 09, 11, 12, 14, 15, 16, 17, 20, 22, 25, 28, 33, 40, 44, 45, 65 をサポートする。決済プロセッサ統合(ZEBEDEE, Nodeless, OpenNode, LNBits, LNURL, NWC)やWeb of Trustフィルタリング、NIP-05検証といった運用機能も備える。

---

### 3. nostr-rs-relay (Rust)

**設定ファイル:** `config.toml`

**デフォルト設定:**
- デフォルト設定に明示的な `maxLimit` または類似のパラメータは見つからず
- 設定はレート制限、接続制限、イベントサイズ制限に焦点

**動作:**
- フィルターに `limit` が指定された場合: その値で SQL LIMIT 句を適用 (`ORDER BY e.created_at DESC LIMIT {lim}`)
- `limit` が指定されていない場合: LIMIT 句を付けず `ORDER BY e.created_at ASC` でクエリする
- **limit未指定時は全てのマッチするイベントを返す** (潜在的に無制限)

**ソースコードの証拠:**
```rust
// src/repo/sqlite.rs:1151-1152
if let Some(lim) = f.limit {
    let _ = write!(query, " ORDER BY e.created_at DESC LIMIT {lim}");
}
```

**サイズ制限 (デフォルト値):**
- イベントサイズ: 131,072バイト (128 KB)
- WebSocketメッセージ: 131,072バイト (128 KB)
- WebSocketフレーム: 131,072バイト (128 KB)

**時間制限:**
- 未来に1,800秒 (30分) を超えるイベントを拒否
- 過去方向の制限や自動削除・保持期間のポリシーは未実装

**レート制限 (設定可能):**
- 秒あたりメッセージ数: 設定可能 (デフォルト: 無制限)
- 分あたりサブスクリプション数: 設定可能 (デフォルト: 無制限)
- サブスクリプション数・REQあたりフィルター数の上限は存在しない

**サポートNIP:** 1, 2, 9, 11, 12, 15, 16, 20, 22, 33, 40 (NIP-42 は `nip42_auth` 有効時のみ追加)

**注目すべき設定オプション:**
```toml
[options]
reject_future_seconds = 1800

[limits]
# messages_per_sec = 5
# subscriptions_per_min = 0
# max_event_bytes = 131072
# max_ws_message_bytes = 131072
# max_ws_frame_bytes = 131072
```

---

### 4. khatru (Goフレームワーク)

**設定:** コードベース、設定ファイルなし

**デフォルト設定:**
- フレームワークに組み込みの最大limit強制なし
- 開発者は独自のlimitポリシーを実装する必要がある

**動作:**
- フレームワークは `LimitZero` フラグを処理して `limit: 0` のクエリをスキップ
- 実際のlimit強制はkhatruを使用するリレー実装に依存
- レート制限ヘルパーを提供するが、結果制限キャップはなし

**フレームワーク機能:**
- カスタムイベント/フィルター受け入れポリシー
- カスタムAUTHハンドラー
- プラガブルストレージバックエンド
- `policies` パッケージの組み込みポリシーヘルパー

**デフォルトサイズ制限:**
- 最大メッセージサイズ: 512,000バイト (500 KB)

**デフォルトレート制限 (`ApplySaneDefaults`経由):**
- イベントレート: 3分あたり2イベント (max 10 tokens)
- フィルターレート: 分あたり20フィルター (max 100 tokens)
- 接続レート: 5分あたり1接続 (max 100 tokens)

**時間制限:**
- フレームワークはデフォルトでは `created_at` を検証しない (未来/過去オフセットの強制なし)
- `policies` パッケージは `PreventTimestampsInTheFuture` / `PreventTimestampsInThePast` ヘルパーを提供するが、`ApplySaneDefaults` には含まれず、利用は実装側の任意

**ソースからの例:**
```go
// responding.go:21-24
if filter.LimitZero {
    return nil, fmt.Errorf("invalid limit 0")
}

// relay.go:45
MaxMessageSize: 512000,
```

---

### haven

havenはGo製のリレーで、khatruフレームワークをベースとしている。最大の特徴は、1つのバイナリで4種類のリレー(Private・Chat・Inbox・Outbox)を同時に提供する点であり、それぞれが独立したレート制限設定を持つ。設定は`.env.example`を通じた環境変数で行われ、`limits.go`の`initRelayLimits()`が各リレーの制限値を初期化する。

**limitパラメータ**: havenはlimitパラメータのデフォルト上限を独自に設定していない。各リレーの`QueryEvents`は`eventstore`ライブラリ(v0.17.5)の実装をそのまま登録しており([init.go#L164](https://github.com/bitvora/haven/blob/8d26f9e/init.go#L164))、khatruの挙動を継承する。クライアントが`limit`を省略した場合、データベースからマッチする全イベントを返す可能性があり、結果サイズのハード上限が存在しない。これは大規模データセットでは潜在的に高負荷となりうる。

**レート制限**: havenの制限はリレータイプ単位で細かく設定されている。イベント送信はPrivate/Chatが50 events/min(max 100 tokens)、Inboxが10 events/min(max 20 tokens)、Outboxが10 events/60min(max 100 tokens)。接続レートは全タイプで3接続/intervalだが、intervalがPrivateは5分、Chatは3分、Inbox/Outboxは1分と異なり、トークン上限(maxTokens)は全タイプ9に統一されている。これらはトークンバケット方式で、`time.Minute * interval`ごとにトークンを補充する。重要なのは、これらのレート制限がリクエスト頻度にのみ適用され、1リクエストが返す結果サイズには適用されない点である。

**時間ベースの制限**: haven自体は`created_at`の未来/過去オフセット検証を実装しておらず、khatruの挙動を継承するためデフォルトでは時刻検証は行われない。別軸として、インポートやWoT構築のためのフェッチタイムアウトが存在し、オーナーノートインポートは60秒、タグ付きノートインポートは120秒、WoTフェッチは30秒がデフォルトとなっている。

**フィルター値・サイズ制限**: フィルター値数やREQあたりフィルター数の独自上限はなく、khatruの`MaxMessageSize`(512,000バイト=500 KB)が実質的な上限として機能する。authorsに換算すると概算で約7,400件となる。リレータイプによっては`AllowEmptyFilters`/`AllowComplexFilters`がfalseに設定され、Chat/Inbox/Outboxでは空フィルターや複雑フィルターが拒否される。

**特別な機能**: 4リレー統合に加え、Blossomメディアサーバー、Web of Trustフィルタリング、クラウドバックアップ(S3互換)、BadgerDBまたはLMDBストレージをサポートする。LMDBのデフォルトマップサイズは約273GBと大きい。

総じてhavenは個人/小規模グループ向けの「全部入り」リレーであり、レート制限はリクエスト頻度に対しては堅牢だが、結果サイズの上限が無いためlimit未指定のクエリには注意が必要である。

---

### 6. wot-relay (Go, Khatruベース)

**設定ファイル:** `.env.example`

**デフォルト設定:**
- khatru フレームワーク (新モノレポ `fiatjaf.com/nostr/khatru`) + eventstore ライブラリを使用
- データベースバックエンド: LMDB (`fiatjaf.com/nostr/eventstore/lmdb`)
- `relay.UseEventstore(db, 500)` により最大クエリ limit を 500 に設定

**動作:**
- khatru フレームワークから動作を継承
- QueryEvents は eventstore (LMDB) バックエンドが処理
- `limit` 未指定時: khatru の maxQueryLimit (=500) が結果サイズの上限として強制される
- ネゲントロピー (NIP-77) セッションのみ maxQueryLimit×20 = 10,000 に拡大されるが、wot-relay は Negentropy を有効化していないため通常 500

**ソースコードの証拠:**
```go
// main.go:160 — LMDB バックエンド
db = &lmdb.LMDBBackend{Path: config.DBPath}
// main.go:164 — 最大クエリ limit を 500 に設定
relay.UseEventstore(db, 500)
```

**レート制限・フィルターポリシー:**
- OnEvent: base64 メディア拒否 → EventIPRateLimiter(5/min, max30) → WoT 外拒否 → 暗号化DM (kind 4) 拒否
- OnRequest: NoEmptyFilters → NoComplexFilters (タグ2個超かつ要素4個超を拒否) → FilterIPRateLimiter(5/min, max30)
- RejectConnection: ConnectionRateLimiter(10/2min, max30)
- khatru デフォルトより厳格。最大サブスクリプション数は未設定 (制限なし)

**特別な機能:**
- Web of Trust リレー (フォローしているユーザーのノートのみ保存)
- 設定可能な WoT 深度と最小フォロワー数 (MINIMUM_FOLLOWERS、MAX_TRUST_NETWORK=40000、MAX_ONE_HOP_NETWORK=50000)
- オプションの他リレーからのアーカイブ同期 (ARCHIVAL_SYNC)、リアクションのアーカイブ可否 (ARCHIVE_REACTIONS)
- オプションの経過時間ベースのノート削除 (ARCHIVE_KINDS 該当 kind)

**サイズ制限:**
- khatru の最大メッセージサイズを継承: 512,000 バイト (500 KB)

**時間制限:**
- created_at の未来・過去検証は登録されておらず強制なし (khatru継承)
- 最大イベント経過時間: `.env.example` 例では 365 日だが、環境変数未設定時のコード上のデフォルトは 0 (削除無効)。MAX_AGE_DAYS で設定。

**サポート NIP:**
- khatru が DeleteEvent 設定で NIP-9、Count 設定で NIP-45 を NIP-11 応答に自動追加。基盤プロトコルとして NIP-1 / NIP-11 をサポート。Negentropy 無効のため NIP-77 は非対応。

---


## 参考文献

- strfry: https://github.com/hoytech/strfry
- nostream: https://github.com/cameri/nostream
- nostr-rs-relay: https://github.com/scsibug/nostr-rs-relay
- khatru: https://github.com/fiatjaf/khatru
- haven: https://github.com/bitvora/haven
- wot-relay: https://github.com/bitvora/wot-relay
- eventstore: https://github.com/fiatjaf/eventstore
- nostr.watch (リレー統計): https://nostr.watch/relays/software

---

## バージョン情報

**ドキュメントバージョン:** 1.4
**Last Checked:** 2026-06-26
**分析したリレーバージョン:**
- strfry: `b80cda3a812af1b662223edad47eb70b053508b6` (2026-06-22)
- nostream: `33f0ba98530d87a1e54ea1bd64481a425294021d` (2026-06-25)
- nostr-rs-relay: `b5c1f642e4f4c3b9c54f5d18d66f4c53642076b4` (2026-05-22)
- khatru: `9f99b9827a6e030bbcefc48f7af68bfe7eea1a27` (2025-09-22)
- haven: `8d26f9e6dfe4f6e43332d30bbf26064675f08559` (2026-06-18)
- wot-relay: `7c5803ff3e765d2b553bce24d8bc2d0a0717fee6` (2026-04-22)
