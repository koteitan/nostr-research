[<< back](../README-en.md) | [Japanese](haven.md) | [English]

# haven

## Overview
- **Language**: Go
- **Based on**: Khatru
- **Config File**: `.env.example`
- **Repository**: https://github.com/bitvora/haven
- **Verified Version**: `81781b36aaafe5895bb612a452b7c0f44cde6e67` (2025-10-26)

## Limit Parameter

**Default Max Limit**: No limit

**Config Parameter**: Not configured

**Behavior**:
- Inherits behavior from Khatru framework
- Uses `eventstore` library for QueryEvents implementation
- When `limit` not specified: **returns all matching events from database**
- **No hard limit on result size** - potentially dangerous for large datasets

**Source Code Evidence**:
```go
// init.go:134
privateRelay.QueryEvents = append(privateRelay.QueryEvents, privateDB.QueryEvents)
// Uses github.com/fiatjaf/eventstore v0.17.1
```

## Rate Limiting

### Rate Limits by Relay Type

| Relay Type | Event Submission Rate | Connection Rate | Source |
|------------|----------------------|-----------------|--------|
| Private | [50 events/min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L61) (max 100 tokens) | [3 conn/5min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L66) | limits.go |
| Chat | [50 events/min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L72) (max 100 tokens) | [3 conn/3min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L77) | limits.go |
| Inbox | [10 events/min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L83) (max 20 tokens) | [3 conn/min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L88) | limits.go |
| Outbox | [10 events/60min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L94) (max 100 tokens) | [3 conn/min](https://github.com/bitvora/haven/blob/81781b36/limits.go#L99) | limits.go |

**Notes**: Max subscriptions and filter/REQ rate not separately configured

## Time-based Restrictions

### Event Timestamp Validation

| Item | Value |
|------|-------|
| Max Future Offset | Not enforced |
| Max Past Offset | Not enforced |

**Notes**: Inherits khatru behavior

### Timeout Settings

| Item | Value |
|------|-------|
| Owner Note Import Fetch Timeout | 30 sec (default) |
| Tagged Note Import Fetch Timeout | 120 sec (default) |
| Web of Trust Fetch Timeout | 30 sec (default) |

## Size Limits

| Item | Value |
|------|-------|
| Max Message Size | Inherits Khatru max message size: 512,000 bytes (500 KB) |

## Special Features

- 4 relays in 1 (Private, Chat, Inbox, Outbox)
- Blossom media server
- Web of Trust filtering
- Cloud backup (S3 compatible)
- BadgerDB or LMDB storage

## Warning

Rate limits apply to request rate, not result size. A single request without limit could still return unlimited events.

---
[<< back](../README-en.md)
