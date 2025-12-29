← [README](../README.md)

# noStrudel Bootstrap Relay

## 結論
- **Bootstrap リレー**: relay.primal.net, relay.damus.io, nos.lol

## ソースコード

**ファイル**: `src/const.ts` (行 45-52)

```typescript
/** The default set of relays to use for fetching users events who have out published outboxes */
export const DEFAULT_FALLBACK_RELAYS = relays(["wss://relay.primal.net/", "wss://relay.damus.io/"]);

/** The default recommended relays to use when a user has not outboxes */
export const RECOMMENDED_FALLBACK_RELAYS = relays([
  "wss://relay.primal.net/",
  "wss://relay.damus.io/",
  "wss://nos.lol/",
]);
```

## 説明

- `DEFAULT_FALLBACK_RELAYS`: outbox が未公開のユーザーのイベント取得用
- `RECOMMENDED_FALLBACK_RELAYS`: outbox がないユーザー向けの推奨リレー
- `DEFAULT_LOOKUP_RELAYS`: `purplepag.es` をユーザー情報取得に使用

## 参考
- https://github.com/hzrd149/nostrudel/blob/main/src/const.ts

---
← [README](../README.md)
