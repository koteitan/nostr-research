[<< back](../README-en.md) | [Japanese](haven.md) | [English]

# haven

## Overview
- **Language**: Go
- **Based on**: Khatru
- **Config File**: `.env.example`
- **Repository**: https://github.com/bitvora/haven
- **Verified Version**: `8d26f9e6dfe4f6e43332d30bbf26064675f08559` (2026-06-18)

## Limit Parameter

**Default Max Limit**: [No limit](https://github.com/bitvora/haven/blob/8d26f9e/init.go#L164) (inherited from Khatru)

**Config Parameter**: Not configured

**Behavior**:
- haven does not define its own default cap for the `limit` parameter
- The `QueryEvents` of each relay (Private/Chat/Inbox/Outbox) registers the `QueryEvents` of the eventstore library (github.com/fiatjaf/eventstore v0.17.5) as-is, inheriting the behavior of the Khatru framework
- When `limit` not specified: **may return all matching events from the database**
- **No hard limit on result size** - potentially heavy load for large datasets

**Source Code Evidence**:
```go
// init.go:164
privateRelay.QueryEvents = append(privateRelay.QueryEvents, privateDB.QueryEvents)
// Uses github.com/fiatjaf/eventstore v0.17.5
```

## Rate Limiting

### Rate Limits by Relay Type

haven serves four relay types (Private/Chat/Inbox/Outbox) from a single binary, each with its own independent event-submission rate (`EventIPRateLimiter`) and connection rate (`ConnectionRateLimiter`). These rates limit request frequency and do not apply to the result size returned by a single request. Max subscriptions and filter/REQ rate are not separately configured on the haven side and inherit Khatru's behavior. They use a token-bucket scheme that refills `tokens` tokens every `interval` (minutes), capped at `maxTokens`. The connection-rate `maxTokens` is 9 for all types.

| Relay Type | Max Subscriptions | Event Submission Rate | Filter/REQ Rate | Connection Rate | Notes | Source |
|------------|-------------------|----------------------|-----------------|-----------------|-------|--------|
| Private | Inherited from Khatru | [50 events/min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L61) (max 100 tokens) | Inherited from Khatru | [3 conn/5min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L66) (max 9 tokens) | Authentication + whitelist required relay | limits.go |
| Chat | Inherited from Khatru | [50 events/min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L72) (max 100 tokens) | Inherited from Khatru | [3 conn/3min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L77) (max 9 tokens) | Chat for users within the WoT | limits.go |
| Inbox | Inherited from Khatru | [10 events/min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L83) (max 20 tokens) | Inherited from Khatru | [3 conn/min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L88) (max 9 tokens) | Inbox relay | limits.go |
| Outbox | Inherited from Khatru | [10 events/60min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L94) (max 100 tokens) | Inherited from Khatru | [3 conn/min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L99) (max 9 tokens) | For public message/media delivery | limits.go |

**Notes**: Max subscriptions and filter/REQ rate are not separately configured and inherit Khatru's behavior

## Time-based Restrictions

### Event Timestamp Validation

| Item | Value |
|------|-------|
| Max Future Offset | Not enforced (inherited from Khatru) |
| Max Past Offset | Not enforced (inherited from Khatru) |

**Notes**: haven itself does not implement future/past offset validation of `created_at`. It inherits the Khatru framework's behavior, so by default no `created_at` validation is performed. Note that these are storage/delivery policies; import/WoT fetching has its own independent timeouts (see below).

### Timeout Settings

| Item | Value |
|------|-------|
| Owner Note Import Fetch Timeout | 60 sec (`IMPORT_OWNER_NOTES_FETCH_TIMEOUT_SECONDS`, default) |
| Tagged Note Import Fetch Timeout | 120 sec (`IMPORT_TAGGED_NOTES_FETCH_TIMEOUT_SECONDS`, default) |
| Web of Trust Fetch Timeout | 30 sec (`WOT_FETCH_TIMEOUT_SECONDS`, default) |

## Filter Value Limits

| Item | Value | Config |
|------|-------|--------|
| Filter Value Limit | No limit (inherited from Khatru) | - |
| Max Filters per REQ | No limit (inherited from Khatru) | - |
| Max authors (approx.) | ~7,400 (due to WebSocket message size limit) | - |

**Notes**: haven does not define its own cap on the number of filter values or filters per REQ. Khatru's `MaxMessageSize` (512,000 bytes) acts as the effective cap, which converts to roughly 7,400 authors. Note that `AllowComplexFilters`/`AllowEmptyFilters` are controlled per relay type; Chat/Inbox/Outbox reject empty and complex filters.

## Size Limits

| Item | Value | Config |
|------|-------|--------|
| Max Message Size | [512,000 bytes (500 KB)](https://github.com/fiatjaf/khatru/blob/v0.19.1/relay.go) (inherited from Khatru) | Khatru MaxMessageSize |

## Supported NIPs

No explicit SupportedNIPs list configured (relies on Khatru/NIP-11 defaults)

## Special Features

- 4 relays in 1 (Private, Chat, Inbox, Outbox)
- Blossom media server
- Web of Trust filtering
- Cloud backup (S3 compatible)
- BadgerDB or LMDB storage (LMDB default map size is approximately 273 GB)

## Warning

Rate limits apply to request frequency, not result size. A single request without `limit` could still return unlimited events.

---
[<< back](../README-en.md)
