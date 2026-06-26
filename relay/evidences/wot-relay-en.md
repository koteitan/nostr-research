[<< back](../README-en.md) | [Japanese](wot-relay.md) | [English]

# wot-relay

## Overview
- **Language**: Go
- **Based on**: Khatru
- **Config File**: `.env.example`
- **Repository**: https://github.com/bitvora/wot-relay
- **Verified Version**: `7c5803ff3e765d2b553bce24d8bc2d0a0717fee6` (2026-04-22)

## Limit Parameter

**Default Max Limit**: [500](https://github.com/bitvora/wot-relay/blob/7c5803f/main.go#L164)

**Config Parameter**: 2nd argument of `UseEventstore` (maxQueryLimit)

**Behavior**:
- The previous version (24b51de9) had no explicit `limit` setting and was therefore "no limit", but this version calls `relay.UseEventstore(db, 500)`, which sets Khatru's maxQueryLimit to 500.
- Khatru's `QueryStored` passes this maxQueryLimit to the store's `QueryEvents` and enforces it as the upper bound on the result size.
- Only for Negentropy (NIP-77) sessions is it expanded to maxQueryLimit × 20 = 10,000, but since wot-relay does not enable Negentropy, the cap is normally 500.
- The eventstore backend was changed from BadgerDB to LMDB (`fiatjaf.com/nostr/eventstore/lmdb`).

**Source Code Evidence**:
```go
relay.UseEventstore(db, 500)  // main.go:164
// khatru: maxLimit := maxQueryLimit (=500); ×20 for negentropy
```

## Rate Limiting

| Item | Value | Source |
|------|-------|--------|
| Max Subscriptions | No limit | - |
| Event Submission Rate | [5 events/min](https://github.com/bitvora/wot-relay/blob/7c5803f/main.go#L168) (max 30 tokens) | main.go |
| Filter Rate | [5 filters/min](https://github.com/bitvora/wot-relay/blob/7c5803f/main.go#L202) (max 30 tokens) | main.go |
| Connection Rate | [10 conn/2min](https://github.com/bitvora/wot-relay/blob/7c5803f/main.go#L205) (max 30 tokens) | main.go |

**Notes**: Khatru-based. Stricter than khatru defaults

Uses the rate-limit token buckets from Khatru's `policies` package. The values are unchanged from the previous version (5 events/min, 5 filters/min, 10 conn/2min, all with max 30 tokens). The maximum number of subscriptions is not explicitly set (inherited from khatru = no limit). `OnEvent` also adds base64 media rejection, non-WoT rejection, and encrypted DM (kind 4) rejection. `OnRequest` adds NoEmptyFilters / NoComplexFilters (rejects filters with more than 2 tags and more than 4 elements).

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

**Notes**: Because Khatru's PreventTimestampsInTheFuture/Past are not registered on `OnEvent`, `created_at` future/past offset validation is not enforced (inherited from khatru = not enforced).

### Event Retention/Deletion Policy

| Item | Value |
|------|-------|
| Regular Event Max Age | Deleted after [365 days](https://github.com/bitvora/wot-relay/blob/7c5803f/.env.example#L27) (although the code default is [0=disabled](https://github.com/bitvora/wot-relay/blob/7c5803f/main.go#L283)) |

**Notes**: Age-based deletion is configured via MAX_AGE_DAYS. The `.env.example` sample uses 365, but the code default when the environment variable is unset is 0 (deletion disabled). When MAX_AGE_DAYS > 0, only the kinds matching ARCHIVE_KINDS are affected.

## Filter Value Limits

| Item | Value | Config |
|------|-------|--------|
| Filter Value Limit | No limit | - |
| Max Filters per REQ | No limit | - |
| Max authors (approx.) | ~7,400 (due to WebSocket message size limit) | - |

**Notes**: Inherited from khatru. There is no limit on the number of filter values themselves, but NoComplexFilters rejects filters with more than 2 tags and more than 4 (tag + kind) elements. The number of authors is effectively capped by the maximum message size of 512,000 bytes (1 pubkey ≈ 69 bytes → about 7,400).

## Size Limits

| Item | Value | Config |
|------|-------|--------|
| Max Message Size | [512,000 (500 KB)](https://github.com/bitvora/wot-relay/blob/7c5803f/main.go#L140) | MaxMessageSize (inherited from khatru) |

**Notes**: The maximum message size inherits Khatru's default of 512,000 bytes (500 KB). wot-relay does not override it (khatru NewRelay's MaxMessageSize: 512000).

## Config Options

```bash
REFRESH_INTERVAL_HOURS=1
MINIMUM_FOLLOWERS=5
ARCHIVAL_SYNC="FALSE"
MAX_AGE_DAYS=365
```

## Special Features

- Web of Trust relay (only stores notes from followed users)
- Configurable WoT depth and minimum followers (MINIMUM_FOLLOWERS, MAX_TRUST_NETWORK=40000, MAX_ONE_HOP_NETWORK=50000)
- Optional archival sync from other relays (ARCHIVAL_SYNC), and whether reactions are archived (ARCHIVE_REACTIONS)
- Optional age-based note deletion (kinds matching ARCHIVE_KINDS)

## Warning

Rate limits apply to request rate, not result size. A single request without `limit` still returns up to the maxQueryLimit of 500 events (10,000 only for Negentropy sessions, which are not enabled here).

---
[<< back](../README-en.md)
