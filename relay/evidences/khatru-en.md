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
- Framework handles `LimitZero` flag to skip queries with `limit: 0`
- Actual limit enforcement depends on relay implementations using Khatru
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

| Item | Value | Source |
|------|-------|--------|
| Max Subscriptions | No limit | - |
| Event Rate | [2 events/3min](https://github.com/fiatjaf/khatru/blob/9f99b982/policies/sane_defaults.go#L12) (max 10 tokens) | [`ApplySaneDefaults`](https://github.com/fiatjaf/khatru/blob/9f99b982/policies/sane_defaults.go#L9) |
| Filter Rate | [20 filters/min](https://github.com/fiatjaf/khatru/blob/9f99b982/policies/sane_defaults.go#L17) (max 100 tokens) | [`ApplySaneDefaults`](https://github.com/fiatjaf/khatru/blob/9f99b982/policies/sane_defaults.go#L9) |
| Connection Rate | [1 conn/5min](https://github.com/fiatjaf/khatru/blob/9f99b982/policies/sane_defaults.go#L21) (max 100 tokens) | [`ApplySaneDefaults`](https://github.com/fiatjaf/khatru/blob/9f99b982/policies/sane_defaults.go#L9) |

## Time-based Restrictions

### Event Timestamp Validation

| Item | Value |
|------|-------|
| Max Future Offset | Not enforced |
| Max Past Offset | Not enforced |

**Notes**: Framework does not enforce by default

## Size Limits

| Item | Value |
|------|-------|
| Max Message Size | 512,000 bytes (500 KB) |

## Framework Features

- Custom event/filter acceptance policies
- Custom AUTH handlers
- Pluggable storage backends
- Built-in policy helpers in `policies` package

---
[<< back](../README-en.md)
