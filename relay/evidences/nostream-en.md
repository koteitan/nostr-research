[<< back](../README-en.md) | [Japanese](nostream.md) | [English]

# nostream

## Overview
- **Language**: TypeScript
- **Config File**: `resources/default-settings.yaml`
- **Repository**: https://github.com/cameri/nostream
- **Verified Version**: `6a8ccb491973b2470e3b332bef61ed7ea1143091` (2024-10-23)

## Limit Parameter

**Default Max Limit**: [5000](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L162)

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
- `maxLimit: 5000` setting controls the maximum allowed limit value
- Also enforces:
  - Max concurrent subscriptions per client: 10
  - Max filters per subscription: 10
  - Max total filter values: 2500

## Rate Limiting

### Rate Limits by Event Kind

| Kind | Event Submission Rate | Description |
|------|----------------------|-------------|
| [0](https://github.com/nostr-protocol/nips/blob/master/01.md), [3](https://github.com/nostr-protocol/nips/blob/master/02.md), [40](https://github.com/nostr-protocol/nips/blob/master/28.md), [41](https://github.com/nostr-protocol/nips/blob/master/28.md) | [6 events/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L108-L115) | Metadata, contacts, channel events |
| [1](https://github.com/nostr-protocol/nips/blob/master/01.md), [2](https://github.com/nostr-protocol/nips/blob/master/01.md), [4](https://github.com/nostr-protocol/nips/blob/master/04.md), [42](https://github.com/nostr-protocol/nips/blob/master/28.md) | [12 events/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L116-L123) | Notes, DMs, channel messages |
| [5](https://github.com/nostr-protocol/nips/blob/master/09.md)-[7](https://github.com/nostr-protocol/nips/blob/master/25.md), [43](https://github.com/nostr-protocol/nips/blob/master/28.md)-[49](https://github.com/nostr-protocol/nips/blob/master/49.md) | [30 events/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L158-L131) | Deletions, reactions, channel events |
| [10000-19999, 30000-39999](https://github.com/nostr-protocol/nips/blob/master/01.md) | [24 events/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L132-L141) | Replaceable events |
| [20000-29999](https://github.com/nostr-protocol/nips/blob/master/01.md) | [60 events/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L142-L147) | Ephemeral events |
| All events | [720 events/hour](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L148-L150) | Overall limit |

### Other Limits

| Item | Value |
|------|-------|
| Max Subscriptions | [10](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L158) |
| Filter/REQ Rate | [240 msg/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L168) |
| Connection Rate | [12/sec](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L71) and [48/min](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L73) |

## Time-based Restrictions

### Event Timestamp Validation

| Item | Value |
|------|-------|
| Max Future Offset | [+900 sec (15 min)](https://github.com/cameri/nostream/blob/6a8ccb49/resources/default-settings.yaml#L90) |
| Max Past Offset | No limit |

## Size Limits

| Item | Value |
|------|-------|
| Network Payload | 524,288 bytes (512 KB) |
| Event Content | 102,400 bytes (100 KB) for kind ranges 0-10, 40-49, 11-39, 50-max |

## Supported NIPs

01, 02, 04, 09, 11, 11a, 12, 13, 15, 16, 20, 22, 28, 33, 40

## Additional Features

- Payment processor integration (ZEBEDEE, Nodeless, OpenNode, LNBits, LNURL)
- Detailed rate limits by event kind

---
[<< back](../README-en.md)
