← [README](../README.md)

# iris Bootstrap Relay

## 結論
- **Bootstrap リレー**: temp.iris.to, vault.iris.to, relay.damus.io, relay.snort.social, nos.lol

## ソースコード

**ファイル**: `src/shared/constants/relays.ts`

```typescript
const PRODUCTION_RELAYS = [
  "wss://temp.iris.to/",
  "wss://vault.iris.to/",
  "wss://relay.damus.io/",
  "wss://relay.snort.social/",
  "wss://nos.lol/",
]

function getDefaultRelays() {
  if (import.meta.env.VITE_USE_TEST_RELAY) {
    return TEST_RELAY
  }
  if (import.meta.env.VITE_USE_LOCAL_RELAY) {
    return LOCAL_RELAY
  }
  return PRODUCTION_RELAYS
}

export const DEFAULT_RELAYS = getDefaultRelays()
```

## 参考
- https://github.com/irislib/iris-client/blob/main/src/shared/constants/relays.ts

---
← [README](../README.md)
