[<< back](../README-en.md) | [Japanese](strfry.md) | [English]

# strfry

## Overview
- **Language**: C++
- **Config File**: `strfry.conf`
- **Repository**: https://github.com/hoytech/strfry
- **Verified Version**: `b80cda3a812af1b662223edad47eb70b053508b6` (2026-06-22)

## Limit Parameter

**Default Max Limit**: [500](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L111)

**Config Parameter**: `relay.maxFilterLimit`

```conf
relay {
    # Maximum records that can be returned per filter
    maxFilterLimit = 500
}
```

**Behavior**:
- When a client requests events with `limit: N`, strfry returns **min(N, 500)** events per filter
- If no limit is specified or limit > 500, strfry caps at 500
- This is a per-filter limit, not per-subscription

## Rate Limiting

| Item | Value | Config |
|------|-------|--------|
| Max Subscriptions | [200](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L120) | `relay.maxSubsPerConnection` |
| Event Submission Rate | Not configured by default | Not configured |
| Filter/REQ Rate | Not configured | Not configured |
| Connection Rate | Not configured | Not configured |

**Notes**: `relay.maxSubsPerConnection` limits the number of concurrent REQs per connection. Event submission/filter/connection rates are not configured in the core settings (handle them with an external reverse proxy, etc.). The max number of subscriptions defaults to 200; there is no core setting for an explicit event submission rate, filter rate, or connection rate.

## Time-based Restrictions

### Event Timestamp Validation

| Item | Value |
|------|-------|
| Max Future Offset | [+900 sec (15 min)](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L24) |
| Max Past Offset | [-94,608,000 sec (~3 years)](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L27) |

### Event Retention/Deletion Policy

| Item | Value |
|------|-------|
| Ephemeral Event Age | Reject if older than [60 sec](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L30) |
| Ephemeral Event TTL | Auto-delete after [300 sec](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L33) |
| Regular Event Max Age | - |

**Notes**: Ephemeral event restrictions apply only to Kind 20000-29999. `rejectEventsNewerThanSeconds`/`rejectEventsOlderThanSeconds` validate the future/past offset of `created_at` and reject out-of-range events.

## Filter Value Limits

| Item | Value | Config |
|------|-------|--------|
| Filter Value Limit | No limit | - |
| Max Filters per REQ | [200](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L99) | `relay.maxReqFilterSize` |
| Max authors (approx.) | ~1,900 (WebSocket limit) | - |

**Notes**: There is no explicit filter value limit; the message size limit such as the WebSocket payload (128 KB) is the effective cap. `relay.maxTagsPerFilter` (default 3) also limits the number of tag filters per filter.

## Size Limits

| Item | Value | Config |
|------|-------|--------|
| Event Size | [65,536 bytes (64 KB)](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L21) | `events.maxEventSize` |
| WebSocket Payload | [131,072 bytes (128 KB)](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L96) | `relay.maxWebsocketPayloadSize` |
| Tag Value Size | [1,024 bytes](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L39) | `events.maxTagValSize` |
| Max Tags | [2,000](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L36) | `events.maxNumTags` |

## Supported NIPs

1, 2, 4, 9, 11, 28, 40, 45, 70, 77

---
[<< back](../README-en.md)
