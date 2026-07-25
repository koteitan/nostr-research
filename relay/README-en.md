[<< back](../README.md) | [Japanese](README-ja.md) | [English]

# Nostr Relay Specifications

## Contents
- [Relay Rankings](#relay-rankings-last-checked-20260626)
- [Limit Parameter Behavior](#limit-parameter-behavior)
- [Rate Limiting](#rate-limiting)
- [Filter Values and Message Size Limits](#filter-values-and-message-size-limits)
- [Time-based Restrictions](#time-based-restrictions)
- [References](#references)
- [Version Information](#version-information)

## Relay Rankings (Last Checked: 2026/06/26)

Source: [nostr.watch](https://nostr.watch/relays/software)

| Name | Relays | Versions | Share | Based-on |
|------|--------|----------|-------|----------|
| hoytech/strfry | 237 | 35 | 26.9% | - |
| ~gheartsfield/nostr-rs-relay | 205 | 10 | 23.3% | - |
| bitvora/haven | 71 | 15 | 8.1% | khatru |
| cameri/nostream | 55 | 8 | 6.3% | - |
| bitvora/wot-relay | 25 | 5 | 2.8% | khatru |
| fiatjaf/khatru | 18 | 3 | 2.0% | - |

## Limit Parameter Behavior

This document compares the behavior of the `limit` filter parameter across different Nostr relay implementations.

### Summary Table

| Relay Implementation | Default Max Limit | Config Parameter | Behavior |
|---------------------|-------------------|------------------|----------|
| [strfry](evidences/strfry-en.md) | [500](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L111) | `relay.maxFilterLimit` | Returns min(client_limit, 500) events per filter |
| [nostream](evidences/nostream-en.md) | [5000](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L221) | `limits.client.subscription.maxLimit` | Returns min(client_limit, 5000) events per subscription |
| [nostr-rs-relay](evidences/nostr-rs-relay-en.md) | No explicit limit | Not found in default config | Returns all matching events when limit not specified |
| [khatru](evidences/khatru-en.md) (framework) | No default limit | Not applicable | Implementation-dependent, no built-in max limit |
| [haven](evidences/haven-en.md) (khatru-based) | [No limit](https://github.com/bitvora/haven/blob/8d26f9e/init.go#L164) | Not configured | Returns all matching events via eventstore |
| [wot-relay](evidences/wot-relay-en.md) (khatru-based) | [500](https://github.com/bitvora/wot-relay/blob/7c5803f/main.go#L164) | `UseEventstore` 2nd arg (maxQueryLimit) | maxQueryLimit set to 500 via khatru |

## Rate Limiting

This table shows the rate limits each relay enforces on client requests.

| Relay | Type | Max Subscriptions | Event Submission Rate | Filter/REQ Rate | Connection Rate | Notes |
|-------|------|-------------------|----------------------|-----------------|-----------------|-------|
| [strfry](evidences/strfry-en.md) | - | [200](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L120) | Not configured by default | Not configured | Not configured | All limits are configurable |
| [nostream](evidences/nostream-en.md) | Kind [0](https://github.com/nostr-protocol/nips/blob/master/01.md),[3](https://github.com/nostr-protocol/nips/blob/master/02.md),[40](https://github.com/nostr-protocol/nips/blob/master/28.md),[41](https://github.com/nostr-protocol/nips/blob/master/28.md) | [10](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L217) | [6 events/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L162-L169) | [240 msg/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L225-L227) | [12/sec](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L117-L118) and [48/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L119-L120) | Metadata, contacts, channel events |
| [nostream](evidences/nostream-en.md) | Kind [1](https://github.com/nostr-protocol/nips/blob/master/01.md),[2](https://github.com/nostr-protocol/nips/blob/master/01.md),[4](https://github.com/nostr-protocol/nips/blob/master/04.md),[42](https://github.com/nostr-protocol/nips/blob/master/28.md) | [10](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L217) | [12 events/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L170-L177) | [240 msg/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L225-L227) | [12/sec](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L117-L118) and [48/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L119-L120) | Notes, DMs, channel messages |
| [nostream](evidences/nostream-en.md) | Kind [5](https://github.com/nostr-protocol/nips/blob/master/09.md)-[7](https://github.com/nostr-protocol/nips/blob/master/25.md),[43](https://github.com/nostr-protocol/nips/blob/master/28.md)-[49](https://github.com/nostr-protocol/nips/blob/master/49.md) | [10](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L217) | [30 events/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L178-L185) | [240 msg/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L225-L227) | [12/sec](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L117-L118) and [48/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L119-L120) | Deletions, reactions, channel events |
| [nostream](evidences/nostream-en.md) | Kind [10000-19999,30000-39999](https://github.com/nostr-protocol/nips/blob/master/01.md) | [10](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L217) | [24 events/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L186-L194) | [240 msg/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L225-L227) | [12/sec](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L117-L118) and [48/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L119-L120) | Replaceable events |
| [nostream](evidences/nostream-en.md) | Kind [445](https://github.com/nostr-protocol/nips/blob/master/01.md) | [10](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L217) | [60 events/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L195-L199) | [240 msg/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L225-L227) | [12/sec](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L117-L118) and [48/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L119-L120) | Marmot group events |
| [nostream](evidences/nostream-en.md) | Kind [20000-29999](https://github.com/nostr-protocol/nips/blob/master/01.md) | [10](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L217) | [60 events/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L200-L205) | [240 msg/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L225-L227) | [12/sec](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L117-L118) and [48/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L119-L120) | Ephemeral events |
| [nostream](evidences/nostream-en.md) | All events | [10](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L217) | [720 events/hour](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L206-L208) overall limit | [240 msg/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L225-L227) | [12/sec](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L117-L118) and [48/min](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L119-L120) | Combined across all kinds |
| [nostr-rs-relay](evidences/nostr-rs-relay-en.md) | - | No limit | Configurable (default: unlimited) | Configurable (default: unlimited) | Not configured | Optional: [messages_per_sec](https://github.com/scsibug/nostr-rs-relay/blob/b5c1f64/config.toml#L115), [subscriptions_per_min](https://github.com/scsibug/nostr-rs-relay/blob/b5c1f64/config.toml#L121) |
| [khatru](evidences/khatru-en.md) | - | No limit | [2 events/3min](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/sane_defaults.go#L12) (max 10 tokens) | [20 filters/min](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/sane_defaults.go#L17) (max 100 tokens) | [1 conn/5min](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/sane_defaults.go#L21) (max 100 tokens) | Framework. Via [`ApplySaneDefaults`](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/sane_defaults.go#L9) |
| [haven](evidences/haven-en.md) | Private | Khatru-inherited | [50 events/min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L61) (max 100 tokens) | Khatru-inherited | [3 conn/5min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L66) (max 9 tokens) | Khatru-based |
| [haven](evidences/haven-en.md) | Chat | Khatru-inherited | [50 events/min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L72) (max 100 tokens) | Khatru-inherited | [3 conn/3min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L77) (max 9 tokens) | Khatru-based |
| [haven](evidences/haven-en.md) | Inbox | Khatru-inherited | [10 events/min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L83) (max 20 tokens) | Khatru-inherited | [3 conn/min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L88) (max 9 tokens) | Khatru-based |
| [haven](evidences/haven-en.md) | Outbox | Khatru-inherited | [10 events/60min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L94) (max 100 tokens) | Khatru-inherited | [3 conn/min](https://github.com/bitvora/haven/blob/8d26f9e/limits.go#L99) (max 9 tokens) | Khatru-based |
| [wot-relay](evidences/wot-relay-en.md) | - | No limit | [5 events/min](https://github.com/bitvora/wot-relay/blob/7c5803f/main.go#L168) (max 30 tokens) | [5 filters/min](https://github.com/bitvora/wot-relay/blob/7c5803f/main.go#L202) (max 30 tokens) | [10 conn/2min](https://github.com/bitvora/wot-relay/blob/7c5803f/main.go#L205) (max 30 tokens) | Khatru-based. Stricter than khatru defaults |

**Important notes:**
- **Max Subscriptions**: Maximum concurrent REQ subscriptions per connection. "No limit" means the relay doesn't enforce a cap (framework-dependent)
- **Token bucket algorithm**: Most relays use token bucket rate limiting where tokens refill over time
- **Rate limits ≠ Result limits**: Rate limits control request frequency, not the number of events returned per request
- **Per-IP vs Per-Connection**: Most implementations apply limits per IP address

---

## Filter Values and Message Size Limits

This table shows the limits on the number of values that can be specified in REQ filters (authors, ids, kinds, #tags, etc.) and message size limits.

### Filter Value Limits

| Relay | Filter Value Limit | Max Filters per REQ | Config Parameter | Notes |
|-------|-------------------|--------------------|--------------------|-------|
| [strfry](evidences/strfry-en.md) | No limit | [200](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L99) | `relay.maxReqFilterSize` | Message size limit only |
| [nostream](evidences/nostream-en.md) | [2500](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L219) (total) | [10](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L218) | `limits.client.subscription.maxFilterValues` | Total of all filter values |
| [nostr-rs-relay](evidences/nostr-rs-relay-en.md) | No limit | No limit | - | Message size limit only |
| [khatru](evidences/khatru-en.md) | No limit | No limit | - | No default limit in framework |
| [haven](evidences/haven-en.md) | No limit | No limit | - | Inherits from khatru |
| [wot-relay](evidences/wot-relay-en.md) | No limit | No limit | - | Inherits from khatru |

### Message Size Limits

| Relay | WebSocket Message | Event Size | Content Size | Config Parameter |
|-------|-------------------|------------|--------------|------------------|
| [strfry](evidences/strfry-en.md) | [128 KB](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L96) | [64 KB](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L21) | - | `relay.maxWebsocketPayloadSize`, `events.maxEventSize` |
| [nostream](evidences/nostream-en.md) | [512 KB](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L77) | - | [100 KB](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L147) (per kind) | `network.maxPayloadSize`, `limits.event.content[].maxLength` |
| [nostr-rs-relay](evidences/nostr-rs-relay-en.md) | [128 KB](https://github.com/scsibug/nostr-rs-relay/blob/b5c1f64/config.toml#L138) | [128 KB](https://github.com/scsibug/nostr-rs-relay/blob/b5c1f64/config.toml#L135) | - | `limits.max_ws_message_bytes`, `limits.max_event_bytes` |
| [khatru](evidences/khatru-en.md) | [500 KB](https://github.com/fiatjaf/khatru/blob/9f99b98/relay.go#L45) | - | - | `MaxMessageSize` |
| [haven](evidences/haven-en.md) | 500 KB | - | - | Inherits from khatru |
| [wot-relay](evidences/wot-relay-en.md) | 500 KB | - | - | Inherits from khatru |

### Practical Maximum for authors Array

Calculated as: 1 pubkey = 64 chars (hex) + ~3 chars (quotes, comma) ≈ 67 bytes:

| Relay | Limiting Factor | Max authors (approx.) |
|-------|-----------------|----------------------|
| nostream | `maxFilterValues: 2500` | **2,500** |
| strfry | WebSocket 128 KB | ~1,900 |
| nostr-rs-relay | WebSocket 128 KB | ~1,900 |
| khatru-based | WebSocket 500 KB | ~7,400 |

**Notes:**
- **nostream**: `maxFilterValues` is the total of all filter values including authors, ids, kinds, #tags
- **Others**: No explicit filter value limit; message size limit is the effective cap
- Too many filter values may impact relay performance

---

## Time-based Restrictions

### Event Submission Time Validation

This table shows how relays validate the `created_at` timestamp when clients submit new events.

| Relay | Max Future Offset | Max Past Offset | Notes |
|-------|------------------|-----------------|-------|
| [strfry](evidences/strfry-en.md) | [+900s (15 min)](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L24) | [-94,608,000s (~3 years)](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L27) | Rejects events outside this range |
| [nostream](evidences/nostream-en.md) | [+900s (15 min)](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L144) | [No limit (0)](https://github.com/cameri/nostream/blob/33f0ba9/resources/default-settings.yaml#L145) | Only future events rejected |
| [nostr-rs-relay](evidences/nostr-rs-relay-en.md) | [+1,800s (30 min)](https://github.com/scsibug/nostr-rs-relay/blob/b5c1f64/config.toml#L105) | No limit | Only future events rejected |
| [khatru](evidences/khatru-en.md) | Not enforced | Not enforced | Framework doesn't enforce by default |
| [haven](evidences/haven-en.md) | Not enforced | Not enforced | Inherits khatru behavior |
| [wot-relay](evidences/wot-relay-en.md) | Not enforced | Not enforced | Inherits khatru behavior |

**Purpose of time validation:**
- **Future offset limit**: Prevents clients from creating events with timestamps too far in the future
- **Past offset limit**: Prevents backdate attacks where malicious users create events with very old timestamps

### Event Storage/Deletion Policies

This table shows how relays manage stored events over time.

| Relay | Ephemeral Event Age | Ephemeral Event Lifetime | Regular Event Max Age | Notes |
|-------|---------------------|--------------------------|----------------------|-------|
| [strfry](evidences/strfry-en.md) | Reject if [>60s](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L30) old | Auto-delete after [300s](https://github.com/hoytech/strfry/blob/b80cda3/strfry.conf#L33) | - | Kind 20000-29999 only |
| [nostream](evidences/nostream-en.md) | - | - | - | No automatic deletion |
| [nostr-rs-relay](evidences/nostr-rs-relay-en.md) | - | - | - | No automatic deletion |
| [khatru](evidences/khatru-en.md) | - | - | - | Implementation-dependent |
| [haven](evidences/haven-en.md) | - | - | - | No automatic deletion |
| [wot-relay](evidences/wot-relay-en.md) | - | - | Delete after [365 days](https://github.com/bitvora/wot-relay/blob/7c5803f/.env.example#L27) (code default [0=disabled](https://github.com/bitvora/wot-relay/blob/7c5803f/main.go#L283)) | Configurable via MAX_AGE_DAYS |

**Key differences:**
- **Ephemeral events** (kind 20000-29999): Temporary events not expected to be stored long-term
- **Regular events**: Standard events that relays typically store indefinitely
- **Auto-deletion**: Some relays automatically remove old events to manage storage

---


## References

- strfry: https://github.com/hoytech/strfry
- nostream: https://github.com/cameri/nostream
- nostr-rs-relay: https://github.com/scsibug/nostr-rs-relay
- khatru: https://github.com/fiatjaf/khatru
- haven: https://github.com/bitvora/haven
- wot-relay: https://github.com/bitvora/wot-relay
- eventstore: https://github.com/fiatjaf/eventstore
- nostr.watch (relay statistics): https://nostr.watch/relays/software

---

## Version Information

**Document Version:** 1.4
**Last Checked:** 2026-06-26
**Relay Versions Analyzed:**
- strfry: `b80cda3a812af1b662223edad47eb70b053508b6` (2026-06-22)
- nostream: `33f0ba98530d87a1e54ea1bd64481a425294021d` (2026-06-25)
- nostr-rs-relay: `b5c1f642e4f4c3b9c54f5d18d66f4c53642076b4` (2026-05-22)
- khatru: `9f99b9827a6e030bbcefc48f7af68bfe7eea1a27` (2025-09-22)
- haven: `8d26f9e6dfe4f6e43332d30bbf26064675f08559` (2026-06-18)
- wot-relay: `7c5803ff3e765d2b553bce24d8bc2d0a0717fee6` (2026-04-22)
