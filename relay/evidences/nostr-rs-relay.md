[<< back](../README-ja.md) | [Japanese] | [English](nostr-rs-relay-en.md)

# nostr-rs-relay

## 概要
- **言語**: Rust
- **設定ファイル**: `config.toml`
- **リポジトリ**: https://github.com/scsibug/nostr-rs-relay
- **確認バージョン**: `b5c1f642e4f4c3b9c54f5d18d66f4c53642076b4` (2026-05-22)

## limit パラメータ

**デフォルト最大limit**: 明示的な制限なし

**設定パラメータ**: デフォルト設定に見つからず

**動作**:
- フィルターに `limit` が指定された場合: その値でSQL LIMIT句を適用 (`ORDER BY e.created_at DESC LIMIT {lim}`)
- `limit` が指定されていない場合: LIMIT句を付けず `ORDER BY e.created_at ASC` で全てのマッチするイベントを返す
- **limit未指定時は全てのマッチするイベントを返す** (潜在的に無制限)

**ソースコードの証拠**:
```rust
// src/repo/sqlite.rs:1151-1152
if let Some(lim) = f.limit {
    let _ = write!(query, " ORDER BY e.created_at DESC LIMIT {lim}");
}
```

## レート制限

| 項目 | 値 | 設定 |
|------|-----|------|
| 最大サブスクリプション数 | 制限なし | - |
| イベント送信レート | 設定可能 (デフォルト: 無制限) | [messages_per_sec](https://github.com/scsibug/nostr-rs-relay/blob/b5c1f64/config.toml#L115) |
| フィルター/REQレート | 設定可能 (デフォルト: 無制限) | [subscriptions_per_min](https://github.com/scsibug/nostr-rs-relay/blob/b5c1f64/config.toml#L121) |
| 接続レート | 未設定 | - |

**備考**: サブスクリプション数・REQあたりフィルター数の上限はソース上に存在しない。messages_per_sec / subscriptions_per_min は未設定または 0 で無制限。接続レート制限の設定項目は存在しない。

## 時間ベースの制限

### イベント送信時刻の検証

| 項目 | 値 |
|------|-----|
| 最大未来オフセット | [+1,800秒 (30分)](https://github.com/scsibug/nostr-rs-relay/blob/b5c1f64/config.toml#L105) |
| 最大過去オフセット | 制限なし |

**備考**: `reject_future_seconds`（デフォルト 1,800 秒）を超えて未来の `created_at` を持つイベントのみ拒否する（src/event.rs の is_valid_timestamp）。過去方向の制限や自動削除・保持期間のポリシーは実装されていない（Retention 構造体は TODO のまま未実装）。

## フィルター値制限

| 項目 | 値 | 設定 |
|------|-----|------|
| フィルター値制限 | 制限なし | - |
| REQあたり最大フィルター数 | 制限なし | - |
| 最大authors数 (概算) | ~1,900 (WebSocket制限による) | - |

**備考**: 明示的なフィルター値制限・REQあたり最大フィルター数の設定や enforcement は存在せず、メッセージサイズ制限（WebSocket 128 KB）が実質的な上限となる。

## サイズ制限

| 項目 | 値 | 設定 |
|------|-----|------|
| イベントサイズ | [131,072バイト (128 KB)](https://github.com/scsibug/nostr-rs-relay/blob/b5c1f64/config.toml#L135) | `limits.max_event_bytes` |
| WebSocketメッセージ | [131,072バイト (128 KB)](https://github.com/scsibug/nostr-rs-relay/blob/b5c1f64/config.toml#L138) | `limits.max_ws_message_bytes` |
| WebSocketフレーム | [131,072バイト (128 KB)](https://github.com/scsibug/nostr-rs-relay/blob/b5c1f64/config.toml#L141) | `limits.max_ws_frame_bytes` |

## 設定例

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

## サポートNIP

1, 2, 9, 11, 12, 15, 16, 20, 22, 33, 40 (NIP-42 は `nip42_auth` 有効時のみ追加)

---
[<< back](../README-ja.md)
