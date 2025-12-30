[<< back](../README-en.md) | [Japanese](strfry.md) | [English]

# strfry

## Overview
- **Language**: C++
- **Config File**: `strfry.conf`
- **Repository**: https://github.com/hoytech/strfry
- **Verified Version**: `542552ab0f5234f808c52c21772b34f6f07bec65` (2025-01-10)

## Limit Parameter

**Default Max Limit**: [500](https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L91)

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
| Max Subscriptions | [20](https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L89) | `relay.maxSubsPerConnection` |
| Event Submission Rate | Not configured by default | Configurable |
| Filter/REQ Rate | Not configured | Configurable |
| Connection Rate | Not configured | Configurable |

**Notes**: All limits are configurable

## Time-based Restrictions

### Event Timestamp Validation

| Item | Value |
|------|-------|
| Max Future Offset | [+900 sec (15 min)](https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L24) |
| Max Past Offset | [-94,608,000 sec (~3 years)](https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L27) |

### Event Retention/Deletion Policy

| Item | Value |
|------|-------|
| Ephemeral Event Age | Reject if older than [60 sec](https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L30) |
| Ephemeral Event TTL | Auto-delete after [300 sec](https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L33) |
| Regular Event Max Age | - |

**Notes**: Kind 20000-29999 only

## Size Limits

| Item | Value |
|------|-------|
| Event Size | 65,536 bytes (64 KB) - normalized JSON |
| WebSocket Payload | 131,072 bytes (128 KB) |
| Tag Value Size | 1,024 bytes |
| Max Filters per REQ | 200 |

## Supported NIPs

1, 2, 4, 9, 11, 22, 28, 40, 70, 77

---
[<< back](../README-en.md)
