[<< back](../README-en.md) | [Japanese](wot-relay.md) | [English]

# wot-relay

## Overview
- **Language**: Go
- **Based on**: Khatru
- **Config File**: `.env.example`
- **Repository**: https://github.com/bitvora/wot-relay
- **Verified Version**: `24b51de9fdd8631f4795be85983666e76f220960` (2025-06-16)

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
// main.go:145
relay.QueryEvents = append(relay.QueryEvents, db.QueryEvents)
// Uses github.com/fiatjaf/eventstore
```

## Rate Limiting

| Item | Value | Source |
|------|-------|--------|
| Max Subscriptions | No limit | - |
| Event Submission Rate | [5 events/min](https://github.com/bitvora/wot-relay/blob/24b51de9/main.go#L131) (max 30 tokens) | main.go |
| Filter Rate | [5 filters/min](https://github.com/bitvora/wot-relay/blob/24b51de9/main.go#L137) (max 30 tokens) | main.go |
| Connection Rate | [10 conn/2min](https://github.com/bitvora/wot-relay/blob/24b51de9/main.go#L141) (max 30 tokens) | main.go |

**Notes**: Khatru-based. Stricter than khatru defaults

**Source Code**:
```go
policies.EventIPRateLimiter(5, time.Minute*1, 30)      // 5 events/min, max 30 tokens
policies.FilterIPRateLimiter(5, time.Minute*1, 30)     // 5 filters/min, max 30 tokens
policies.ConnectionRateLimiter(10, time.Minute*2, 30)  // 10 connections/2min, max 30 tokens
```

## Time-based Restrictions

### Event Timestamp Validation

| Item | Value |
|------|-------|
| Max Future Offset | Not enforced |
| Max Past Offset | Not enforced |

**Notes**: Inherits khatru behavior

### Event Retention/Deletion Policy

| Item | Value |
|------|-------|
| Regular Event Max Age | Deleted after [365 days](https://github.com/bitvora/wot-relay/blob/24b51de9/.env.example#L27) |

**Notes**: Configurable via MAX_AGE_DAYS

## Size Limits

| Item | Value |
|------|-------|
| Max Message Size | Inherits Khatru max message size: 512,000 bytes (500 KB) |

## Config Options

```bash
REFRESH_INTERVAL_HOURS=1
MINIMUM_FOLLOWERS=5
ARCHIVAL_SYNC="FALSE"
MAX_AGE_DAYS=365
```

## Special Features

- Web of Trust relay (only stores notes from followed users)
- Configurable WoT depth and minimum followers
- Optional archival sync from other relays
- Optional age-based note deletion

## Warning

Rate limits apply to request rate, not result size. A single request without limit could still return unlimited events.

---
[<< back](../README-en.md)
