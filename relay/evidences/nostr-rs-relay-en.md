[<< back](../README-en.md) | [Japanese](nostr-rs-relay.md) | [English]

# nostr-rs-relay

## Overview
- **Language**: Rust
- **Config File**: `config.toml`
- **Repository**: https://github.com/scsibug/nostr-rs-relay
- **Verified Version**: `b5c1f642e4f4c3b9c54f5d18d66f4c53642076b4` (2026-05-22)

## Limit Parameter

**Default Max Limit**: No explicit limit

**Config Parameter**: Not found in default config

**Behavior**:
- When `limit` is specified in filter: applies SQL LIMIT clause with that value (`ORDER BY e.created_at DESC LIMIT {lim}`)
- When `limit` is not specified: no LIMIT clause is added and the query uses `ORDER BY e.created_at ASC` to return all matching events (potentially unbounded)
- **Returns all matching events when limit not specified** (potentially unbounded). There is no default maximum limit value.

**Source Code Evidence**:
```rust
// src/repo/sqlite.rs:1151-1152
if let Some(lim) = f.limit {
    let _ = write!(query, " ORDER BY e.created_at DESC LIMIT {lim}");
}
```

## Rate Limiting

| Item | Value | Config |
|------|-------|--------|
| Max Subscriptions | No limit | - |
| Event Submission Rate | Configurable (default: unlimited) | [messages_per_sec](https://github.com/scsibug/nostr-rs-relay/blob/b5c1f64/config.toml#L115) |
| Filter/REQ Rate | Configurable (default: unlimited) | [subscriptions_per_min](https://github.com/scsibug/nostr-rs-relay/blob/b5c1f64/config.toml#L121) |
| Connection Rate | Not configured | - |

**Notes**: There is no source-level cap on the number of subscriptions or filters per REQ. The event submission rate and subscription creation rate are configurable, but both are unlimited by default (they are relay-wide aggregate values, not per-connection). `messages_per_sec` / `subscriptions_per_min` are unlimited when unset or 0. No connection-rate-limit setting exists.

## Time-based Restrictions

### Event Timestamp Validation

| Item | Value |
|------|-------|
| Max Future Offset | [+1,800 sec (30 min)](https://github.com/scsibug/nostr-rs-relay/blob/b5c1f64/config.toml#L105) |
| Max Past Offset | No limit |

**Notes**: Only events whose `created_at` is more than `reject_future_seconds` (default 1,800 sec) in the future are rejected (`is_valid_timestamp` in src/event.rs). No past-direction restriction and no automatic-deletion or retention-period policy is implemented (the Retention struct remains a TODO).

## Filter Value Limits

| Item | Value | Config |
|------|-------|--------|
| Filter Value Limit | No limit | - |
| Max Filters per REQ | No limit | - |
| Max authors (approx.) | ~1,900 (WebSocket limit) | - |

**Notes**: There is no explicit filter-value limit or max-filters-per-REQ setting or enforcement; the message size limit (WebSocket 128 KB) is the effective cap.

## Size Limits

| Item | Value | Config |
|------|-------|--------|
| Event Size | [131,072 bytes (128 KB)](https://github.com/scsibug/nostr-rs-relay/blob/b5c1f64/config.toml#L135) | `limits.max_event_bytes` |
| WebSocket Message | [131,072 bytes (128 KB)](https://github.com/scsibug/nostr-rs-relay/blob/b5c1f64/config.toml#L138) | `limits.max_ws_message_bytes` |
| WebSocket Frame | [131,072 bytes (128 KB)](https://github.com/scsibug/nostr-rs-relay/blob/b5c1f64/config.toml#L141) | `limits.max_ws_frame_bytes` |

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

1, 2, 9, 11, 12, 15, 16, 20, 22, 33, 40 (NIP-42 is added only when `nip42_auth` is enabled)

---
[<< back](../README-en.md)
