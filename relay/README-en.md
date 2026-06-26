[<< back](../README.md) | [Japanese](README-ja.md) | [English]

# Nostr Relay Specifications

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


## Detailed Analysis

### 1. strfry (C++)

**Configuration File:** `strfry.conf`

**Default Setting:**
```conf
relay {
    # Maximum records that can be returned per filter
    maxFilterLimit = 500
}
```

**Behavior:**
- When a client requests events with `limit: N`, strfry returns **min(N, 500)** events per filter
- If client doesn't specify limit or specifies limit > 500, strfry caps it at 500
- This is a per-filter limit, not per-subscription limit

**Rate Limits:**
- Max subscriptions (concurrent REQ per connection): 200 (`relay.maxSubsPerConnection`)
- Event submission rate / filter rate / connection rate are not configured as core settings

**Size Limits:**
- Event size: 65,536 bytes (64 KB) - normalized JSON
- WebSocket payload: 131,072 bytes (128 KB)
- Tag value size: 1,024 bytes
- Max number of tags: 2,000
- Max filters per REQ: 200

**Time Limits:**
- Reject events newer than 900 seconds (15 minutes) in the future
- Reject events older than 94,608,000 seconds (~3 years)
- Reject ephemeral events older than 60 seconds
- Ephemeral events lifetime: 300 seconds (5 minutes)

**NIPs Supported:** 1, 2, 4, 9, 11, 28, 40, 45, 70, 77 (NIP-45 when COUNT is enabled, NIP-77 when negentropy is enabled, NIP-42 added when AUTH `serviceUrl` is configured)

---

### 2. nostream (TypeScript)

**Configuration File:** `resources/default-settings.yaml`

**Default Setting:**
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

**Behavior:**
- When a client requests events with `limit: N`, nostream returns **min(N, 5000)** events
- The `maxLimit: 5000` setting controls the maximum allowed limit value
- Also enforces:
  - Maximum 10 concurrent subscriptions per client
  - Maximum 10 filters per subscription
  - Maximum 2500 filter values total

**Size Limits:**
- Network payload: 524,288 bytes (512 KB)
- Event content: 102,400 bytes (100 KB) for kind ranges 0-10, 40-49, and 11-39, 50-max

**Time Limits:**
- Event created_at: maximum 900 seconds (15 minutes) in the future
- Event created_at: maximum 0 seconds in the past (`maxNegativeDelta: 0` disables the past restriction, i.e. no limit)

**Rate Limits (per event kind):**
- Kind 0, 3, 40, 41: 6 events/minute
- Kind 1, 2, 4, 42: 12 events/minute
- Kind 5-7, 43-49: 30 events/minute
- Kind 10000-19999, 30000-39999: 24 events/minute (replaceable)
- Kind 445: 60 events/minute (Marmot group events)
- Kind 20000-29999: 60 events/minute (ephemeral)
- All events: 720 events/hour

**NIPs Supported:** 1, 2, 3, 4, 9, 11, 12, 14, 15, 16, 17, 20, 22, 25, 28, 33, 40, 44, 45, 65

**Additional Features:**
- Payment processor integration (ZEBEDEE, Nodeless, OpenNode, LNBits, LNURL, NWC)
- Web of Trust filtering, NIP-05 verification
- Extensive rate limiting per event kind (EWMA strategy)

---

### 3. nostr-rs-relay (Rust)

**Configuration File:** `config.toml`

**Default Setting:**
- No explicit `maxLimit` or similar parameter found in default configuration
- Configuration focuses on rate limiting, connection limits, and event size limits

**Behavior:**
- When `limit` is specified in filter: applies SQL LIMIT clause with that value (`ORDER BY e.created_at DESC LIMIT {lim}`)
- When `limit` is NOT specified: no LIMIT clause added, queries with `ORDER BY e.created_at ASC`
- **Returns all matching events when limit not specified** (potentially unlimited)

**Source Code Evidence:**
```rust
// src/repo/sqlite.rs:1151-1152
if let Some(lim) = f.limit {
    let _ = write!(query, " ORDER BY e.created_at DESC LIMIT {lim}");
}
```

**Size Limits (default values):**
- Event size: 131,072 bytes (128 KB)
- WebSocket message: 131,072 bytes (128 KB)
- WebSocket frame: 131,072 bytes (128 KB)

**Time Limits:**
- Reject events more than 1,800 seconds (30 minutes) in the future
- No past-direction restriction or auto-deletion/retention policy implemented

**Rate Limits (configurable):**
- Messages per second: configurable (default: unlimited)
- Subscriptions per minute: configurable (default: unlimited)
- No cap on subscription count or filters per REQ

**NIPs Supported:** 1, 2, 9, 11, 12, 15, 16, 20, 22, 33, 40 (NIP-42 added only when `nip42_auth` is enabled)

**Notable Configuration Options:**
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

---

### 4. khatru (Go Framework)

**Configuration:** Code-based, no config file

**Default Setting:**
- No built-in maximum limit enforcement in the framework
- Developers must implement their own limit policies

**Behavior:**
- Framework handles `LimitZero` flag to skip queries with `limit: 0`
- Actual limit enforcement depends on the relay implementation using Khatru
- Provides rate limiting helpers but not result limit caps

**Framework Features:**
- Custom event/filter acceptance policies
- Custom AUTH handlers
- Pluggable storage backends
- Built-in policy helpers in `policies` package

**Default Size Limits:**
- Max message size: 512,000 bytes (500 KB)

**Default Rate Limits (via `ApplySaneDefaults`):**
- Event rate: 2 events per 3 minutes (max 10 tokens)
- Filter rate: 20 filters per minute (max 100 tokens)
- Connection rate: 1 connection per 5 minutes (max 100 tokens)

**Time Limits:**
- Framework does not validate `created_at` by default (no future/past offset enforcement)
- The `policies` package provides `PreventTimestampsInTheFuture` / `PreventTimestampsInThePast` helpers, but they are not included in `ApplySaneDefaults`

**Example from source:**
```go
// responding.go:21-24
if filter.LimitZero {
    return nil, fmt.Errorf("invalid limit 0")
}

// relay.go:45
MaxMessageSize: 512000,
```

---

### 5. haven (Go, Khatru-based)

**Configuration File:** `.env.example`

**Default Setting:**
- No explicit limit configuration found
- Uses Khatru framework + eventstore library (v0.17.5)
- Database backend: BadgerDB or LMDB

**Behavior:**
- Inherits behavior from Khatru framework
- Uses `eventstore` library for QueryEvents implementation
- When `limit` NOT specified: **returns all matching events from database**
- **No hard cap on result size** - potentially dangerous for large datasets

**Source Code Evidence:**
```go
// init.go:164
privateRelay.QueryEvents = append(privateRelay.QueryEvents, privateDB.QueryEvents)
// Uses github.com/fiatjaf/eventstore v0.17.5
```

**Special Features:**
- Four relays in one (Private, Chat, Inbox, Outbox)
- Blossom media server
- Web of Trust filtering
- Cloud backups (S3-compatible)
- BadgerDB or LMDB storage (LMDB default map size ~273 GB)

**Size Limits:**
- Inherits Khatru's max message size: 512,000 bytes (500 KB)

**Time Limits:**
- Import owner notes fetch timeout: 60 seconds (default)
- Import tagged notes fetch timeout: 120 seconds (default)
- Web of Trust fetch timeout: 30 seconds (default)

**Rate Limiting (per relay type):**
- Private: 50 events/min, max 100 tokens; 3 conn/5min, max 9 tokens
- Chat: 50 events/min, max 100 tokens; 3 conn/3min, max 9 tokens
- Inbox: 10 events/min, max 20 tokens; 3 conn/min, max 9 tokens
- Outbox: 10 events/60min, max 100 tokens; 3 conn/min, max 9 tokens

**Warning:** Rate limiting applies to request rate, NOT result size. A single request without limit can still return unlimited events.

---

### 6. wot-relay (Go, Khatru-based)

**Configuration File:** `.env.example`

**Default Setting:**
- Uses Khatru framework (new monorepo `fiatjaf.com/nostr/khatru`) + eventstore library
- Database backend: LMDB (`fiatjaf.com/nostr/eventstore/lmdb`)
- `relay.UseEventstore(db, 500)` sets the max query limit to 500

**Behavior:**
- Inherits behavior from Khatru framework
- QueryEvents handled by eventstore (LMDB) backend
- When `limit` NOT specified: khatru's maxQueryLimit (=500) is enforced as the result size cap
- Only negentropy (NIP-77) sessions expand to maxQueryLimit×20 = 10,000, but wot-relay does not enable Negentropy so the cap is normally 500

**Source Code Evidence:**
```go
// main.go:160 — LMDB backend
db = &lmdb.LMDBBackend{Path: config.DBPath}
// main.go:164 — set max query limit to 500
relay.UseEventstore(db, 500)
```

**Rate Limiting / Filter Policies:**
- OnEvent: reject base64 media → EventIPRateLimiter(5/min, max30) → reject outside WoT → reject encrypted DM (kind 4)
- OnRequest: NoEmptyFilters → NoComplexFilters (reject >2 tags and >4 elements) → FilterIPRateLimiter(5/min, max30)
- RejectConnection: ConnectionRateLimiter(10/2min, max30)
- Stricter than khatru defaults. Max subscriptions not configured (no limit)

**Special Features:**
- Web of Trust relay (only stores notes from followed users)
- Configurable WoT depth and minimum followers (MINIMUM_FOLLOWERS, MAX_TRUST_NETWORK=40000, MAX_ONE_HOP_NETWORK=50000)
- Optional archival sync from other relays (ARCHIVAL_SYNC), reaction archiving (ARCHIVE_REACTIONS)
- Optional age-based note deletion (ARCHIVE_KINDS)

**Size Limits:**
- Inherits Khatru's max message size: 512,000 bytes (500 KB)

**Time Limits:**
- created_at future/past validation not registered (inherits khatru = not enforced)
- Maximum event age: `.env.example` example shows 365 days, but the code default when the env var is unset is 0 (deletion disabled). Configurable via MAX_AGE_DAYS

**NIPs Supported:**
- khatru auto-adds NIP-9 (DeleteEvent) and NIP-45 (Count) to the NIP-11 response. Supports NIP-1 / NIP-11 as the base protocol. NIP-77 unsupported (Negentropy disabled)

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
