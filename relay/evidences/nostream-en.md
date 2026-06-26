[<< back](../README-en.md) | [Japanese](nostream.md) | [English]

# nostream

## Overview
- **Language**: TypeScript
- **Config File**: `resources/default-settings.yaml`
- **Repository**: https://github.com/cameri/nostream
- **Verified Version**: `33f0ba98530d87a1e54ea1bd64481a425294021d` (2026-06-25)

## Limit Parameter

**Default Max Limit**: [5000](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L221)

**Config Parameter**: `limits.client.subscription.maxLimit`

```yaml
limits:
  client:
    subscription:
      maxSubscriptions: 10
      maxFilters: 10
      maxFilterValues: 2500
      maxSubscriptionIdLength: 256
      maxLimit: 5000
      minPrefixLength: 4
```

**Behavior**:
- When a client requests events with `limit: N`, nostream returns **min(N, 5000)** events
- The `maxLimit: 5000` setting controls the maximum allowed limit value
- Also enforces:
  - Max concurrent subscriptions per client: 10
  - Max filters per subscription: 10
  - Max total filter values: 2500

## Rate Limiting

Event submission rates are configured granularly per event kind. In this version, a row for Marmot group events (kind 445) at 60 events/min was newly added. The rate-limiting strategy is `ewma` (exponentially weighted moving average).

### Rate Limits by Event Kind

| Kind | Event Submission Rate | Description |
|------|----------------------|-------------|
| [0](https://github.com/nostr-protocol/nips/blob/master/01.md), [3](https://github.com/nostr-protocol/nips/blob/master/02.md), [40](https://github.com/nostr-protocol/nips/blob/master/28.md), [41](https://github.com/nostr-protocol/nips/blob/master/28.md) | [6 events/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L162-L169) | Metadata, contacts, channel creation/update events |
| [1](https://github.com/nostr-protocol/nips/blob/master/01.md), [2](https://github.com/nostr-protocol/nips/blob/master/01.md), [4](https://github.com/nostr-protocol/nips/blob/master/04.md), [42](https://github.com/nostr-protocol/nips/blob/master/28.md) | [12 events/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L170-L177) | Notes, DMs, channel messages |
| [5](https://github.com/nostr-protocol/nips/blob/master/09.md)-[7](https://github.com/nostr-protocol/nips/blob/master/25.md), [43](https://github.com/nostr-protocol/nips/blob/master/28.md)-[49](https://github.com/nostr-protocol/nips/blob/master/49.md) | [30 events/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L178-L185) | Deletions, reactions, channel events |
| [10000-19999, 30000-39999](https://github.com/nostr-protocol/nips/blob/master/01.md) | [24 events/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L186-L194) | Replaceable / parameterized replaceable events |
| [445](https://github.com/nostr-protocol/nips/blob/master/01.md) | [60 events/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L195-L199) | Marmot group events (newly added) |
| [20000-29999](https://github.com/nostr-protocol/nips/blob/master/01.md) | [60 events/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L200-L205) | Ephemeral events |
| All events | [720 events/hour](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L206-L208) | Overall limit |

### Other Limits

| Item | Value |
|------|-------|
| Max Subscriptions | [10](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L217) |
| Filter/REQ Rate | [240 msg/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L225-L227) |
| Connection Rate | [12/sec](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L117-L118) and [48/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L119-L120) |

**Notes**: Max subscriptions, raw message (including REQ) rate, and connection rate.

## Time-based Restrictions

### Event Timestamp Validation

| Item | Value |
|------|-------|
| Max Future Offset | [+900 sec (15 min)](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L144) |
| Max Past Offset | [No limit (0)](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L145) |

**Notes**: `createdAt.maxPositiveDelta: 900` allows events up to 15 minutes into the future. `maxNegativeDelta: 0` disables the past-direction restriction (meaning no limit). Ephemeral events (kind 20000-29999) are not stored in the DB and are only delivered. The retention period for normal events is unlimited via `event.retention.maxDays: -1`.

## Filter Value Limits

| Item | Value | Config |
|------|-------|--------|
| Filter Value Limit | [2500](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L219) (total) | `limits.client.subscription.maxFilterValues` |
| Max Filters per REQ | [10](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L218) | `limits.client.subscription.maxFilters` |
| Max authors | 2,500 (filter value limit) | - |
| Min Prefix Length | [4](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L222) | `limits.client.subscription.minPrefixLength` |

**Notes**: `maxFilterValues` is the total cap across all filter values including authors, ids, kinds, and #tags. The minimum prefix length is [4](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L222) (`minPrefixLength`). The maximum subscription ID length is [256](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L220).

## Size Limits

| Item | Value | Config |
|------|-------|--------|
| Network Payload | [524,288 bytes (512 KB)](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L77) | `network.maxPayloadSize` |
| Event Content | [102,400 bytes (100 KB)](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L147) | `limits.event.content[].maxLength` |

**Notes**: Content size is configurable per kind range (0-10/40-49, 11-39/50-max).

## Supported NIPs

1, 2, 3, 4, 9, 11, 12, 14, 15, 16, 17, 20, 22, 25, 28, 33, 40, 44, 45, 65

## Additional Features

- Payment processor integration (ZEBEDEE, Nodeless, OpenNode, LNBits, LNURL, NWC)
- Web of Trust filtering
- NIP-05 verification
- Detailed rate limits by event kind

---
[<< back](../README-en.md)
