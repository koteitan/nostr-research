[<< back](../README-ja.md) | [Japanese] | [English](README-en.md)

# Nostr リレー仕様

## 目次
- [リレーランキング](#リレーランキング-last-checked-20260626)
- [limit パラメータの動作](#limit-パラメータの動作)
- [レート制限](#レート制限)
- [フィルター値とメッセージサイズ制限](#フィルター値とメッセージサイズ制限)
- [時間ベースの制限](#時間ベースの制限)
- [参考文献](#参考文献)
- [バージョン情報](#バージョン情報)

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


## 参考文献

- strfry: https://github.com/hoytech/strfry
- nostream: https://github.com/cameri/nostream
- nostr-rs-relay: https://github.com/scsibug/nostr-rs-relay
- khatru: https://github.com/fiatjaf/khatru
- haven: https://github.com/bitvora/haven
- wot-relay: https://github.com/bitvora/wot-relay
- eventstore: https://github.com/fiatjaf/eventstore
- nostr.watch — リレー統計と実装別のサポートNIP一覧: https://nostr.watch/relays/software

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
