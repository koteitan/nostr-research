[<< back](../README-ja.md) | [Japanese] | [English](nostr-rs-relay-en.md)

# nostr-rs-relay

## 概要
- **言語**: Rust
- **設定ファイル**: `config.toml`
- **リポジトリ**: https://git.sr.ht/~gheartsfield/nostr-rs-relay
- **確認バージョン**: `d72af96d5f884e916cd3374dddd5550f8a45bfaf` (2025-02-23)

## limit パラメータ

**デフォルト最大limit**: 明示的な制限なし

**設定パラメータ**: デフォルト設定に見つからず

**動作**:
- フィルターに `limit` が指定された場合: その値でSQL LIMIT句を適用
- `limit` が指定されていない場合: SQLクエリにLIMIT句を追加しない
- **limit未指定時は全てのマッチするイベントを返す** (潜在的に無制限)

**ソースコードの証拠**:
```rust
// src/repo/sqlite.rs:1093-1094
if let Some(lim) = f.limit {
    let _ = write!(query, " ORDER BY e.created_at DESC LIMIT {lim}");
}
```

## レート制限

| 項目 | 値 | 設定 |
|------|-----|------|
| 最大サブスクリプション数 | 制限なし | - |
| イベント送信レート | 設定可能 (デフォルト: 無制限) | [messages_per_sec](https://git.sr.ht/~gheartsfield/nostr-rs-relay/tree/d72af96d/item/config.toml#L115) |
| フィルター/REQレート | 設定可能 (デフォルト: 無制限) | [subscriptions_per_min](https://git.sr.ht/~gheartsfield/nostr-rs-relay/tree/d72af96d/item/config.toml#L121) |
| 接続レート | 未設定 | - |

## 時間ベースの制限

### イベント送信時刻の検証

| 項目 | 値 |
|------|-----|
| 最大未来オフセット | [+1,800秒 (30分)](https://git.sr.ht/~gheartsfield/nostr-rs-relay/tree/d72af96d/item/config.toml#L105) |
| 最大過去オフセット | 制限なし |

## サイズ制限

| 項目 | 値 |
|------|-----|
| イベントサイズ | 131,072バイト (128 KB) |
| WebSocketメッセージ | 131,072バイト (128 KB) |
| WebSocketフレーム | 131,072バイト (128 KB) |

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

01, 02, 05, 09, 11, 12, 15, 16, 20, 22, 28, 33, 40, 42

---
[<< back](../README-ja.md)
