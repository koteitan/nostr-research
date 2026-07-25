[<< back](../README-en.md) | [Japanese](khatru.md) | [English]

# khatru

## Overview
- **Language**: Go
- **Type**: Framework
- **Config**: Codebase, no config file
- **Repository**: https://github.com/fiatjaf/khatru
- **Verified Version**: `9f99b9827a6e030bbcefc48f7af68bfe7eea1a27` (2025-09-22)

## Limit Parameter

**Default Max Limit**: No default limit

**Config Parameter**: Not applicable

**Behavior**:
- No built-in max limit enforcement in the framework
- Developers need to implement their own limit policies
- Framework handles the `LimitZero` flag to skip queries with `limit: 0`
- Actual limit enforcement depends on the relay implementation using khatru
- Provides rate limiting helpers but no result cap

**Source Code Evidence**:
```go
// responding.go:21-24
if filter.LimitZero {
    return nil, fmt.Errorf("invalid limit 0")
}

// relay.go:45
MaxMessageSize: 512000,
```

## Rate Limiting

### Default Rate Limits (via `ApplySaneDefaults`)

Framework. Default rate limits take effect when [`ApplySaneDefaults`](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/sane_defaults.go#L9) is applied. Max subscriptions is not enforced at the framework level.

| Item | Value | Source |
|------|-------|--------|
| Max Subscriptions | No limit | - |
| Event Rate | [2 events/3min](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/sane_defaults.go#L12) (max 10 tokens) | [`ApplySaneDefaults`](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/sane_defaults.go#L9) |
| Filter Rate | [20 filters/min](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/sane_defaults.go#L17) (max 100 tokens) | [`ApplySaneDefaults`](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/sane_defaults.go#L9) |
| Connection Rate | [1 conn/5min](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/sane_defaults.go#L21) (max 100 tokens) | [`ApplySaneDefaults`](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/sane_defaults.go#L9) |

## Time-based Restrictions

### Event Timestamp Validation

| Item | Value |
|------|-------|
| Max Future Offset | Not enforced |
| Max Past Offset | Not enforced |
| Ephemeral Age | - |
| Ephemeral Lifetime | - |
| Normal Max Age | - |

**Notes**: The framework does not validate `created_at` by default. The `policies` package provides [`PreventTimestampsInTheFuture`](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/events.go#L98) / [`PreventTimestampsInThePast`](https://github.com/fiatjaf/khatru/blob/9f99b98/policies/events.go#L88) helpers, but they are not applied by `ApplySaneDefaults`. Event storage/deletion policies are implementation-dependent.

## Filter Value Limits

| Item | Value | Config |
|------|-------|--------|
| Filter Value Limit | No limit | - |
| Max Filters per REQ | No limit | - |
| Max authors (approx.) | ~7,400 (WebSocket limit) | - |

**Notes**: No default filter value limit in the framework. The message size limit ([512,000 bytes (500 KB)](https://github.com/fiatjaf/khatru/blob/9f99b98/relay.go#L45)) is the effective cap.

## Size Limits

| Item | Value | Config |
|------|-------|--------|
| Max Message Size | [512,000 bytes (500 KB)](https://github.com/fiatjaf/khatru/blob/9f99b98/relay.go#L45) | `MaxMessageSize` |

## Supported NIPs

Not applicable (no specific NIP list because it is a framework)

## Framework Features

- Custom event/filter acceptance policies
- Custom AUTH handlers
- Pluggable storage backends
- Built-in policy helpers in the `policies` package

---
[<< back](../README-en.md)
