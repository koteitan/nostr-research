[<< back](../README-en.md) | [Japanese](nostr-rs-relay.md) | [English]

# nostr-rs-relay

## Overview
- **Language**: Rust
- **Config File**: `config.toml`
- **Repository**: https://github.com/scsibug/nostr-rs-relay
- **Verified Version**: `d72af96d5f884e916cd3374dddd5550f8a45bfaf` (2025-02-23)

## Limit Parameter

**Default Max Limit**: No explicit limit

**Config Parameter**: Not found in default config

**Behavior**:
- When `limit` is specified in filter: applies SQL LIMIT clause with that value
- When `limit` is not specified: no LIMIT clause is added to SQL query
- **Returns all matching events when limit not specified** (potentially unbounded)

**Source Code Evidence**:
```rust
// src/repo/sqlite.rs:1093-1094
if let Some(lim) = f.limit {
    let _ = write!(query, " ORDER BY e.created_at DESC LIMIT {lim}");
}
```

## Rate Limiting

| Item | Value | Config |
|------|-------|--------|
| Max Subscriptions | No limit | - |
| Event Submission Rate | Configurable (default: unlimited) | [messages_per_sec](https://github.com/scsibug/nostr-rs-relay/tree/d72af96d/item/config.toml#L115) |
| Filter/REQ Rate | Configurable (default: unlimited) | [subscriptions_per_min](https://github.com/scsibug/nostr-rs-relay/tree/d72af96d/item/config.toml#L121) |
| Connection Rate | Not configured | - |

## Time-based Restrictions

### Event Timestamp Validation

| Item | Value |
|------|-------|
| Max Future Offset | [+1,800 sec (30 min)](https://github.com/scsibug/nostr-rs-relay/tree/d72af96d/item/config.toml#L105) |
| Max Past Offset | No limit |

## Filter Value Limits

| Item | Value | Config |
|------|-------|--------|
| Filter Value Limit | No limit | - |
| Max Filters per REQ | No limit | - |
| Max authors (approx.) | ~1,900 (WebSocket limit) | - |

**Notes**: No explicit filter value limit; message size is the effective cap

## Size Limits

| Item | Value | Config |
|------|-------|--------|
| Event Size | [131,072 bytes (128 KB)](https://github.com/scsibug/nostr-rs-relay/tree/d72af96d/item/config.toml#L136) | `limits.max_event_bytes` |
| WebSocket Message | [131,072 bytes (128 KB)](https://github.com/scsibug/nostr-rs-relay/tree/d72af96d/item/config.toml#L138) | `limits.max_ws_message_bytes` |
| WebSocket Frame | [131,072 bytes (128 KB)](https://github.com/scsibug/nostr-rs-relay/tree/d72af96d/item/config.toml#L140) | `limits.max_ws_frame_bytes` |

## Config Example

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

## Supported NIPs

01, 02, 05, 09, 11, 12, 15, 16, 20, 22, 28, 33, 40, 42

---
[<< back](../README-en.md)
