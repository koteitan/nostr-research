← [README](../README.md)

# noStrudel Bootstrap Relay

## 結論
- **Bootstrap リレー**: ハードコードされたフォールバックリレー。outbox未公開ユーザー向け DEFAULT_FALLBACK_RELAYS は relay.primal.net, relay.damus.io。outbox無しユーザー推奨の RECOMMENDED_FALLBACK_RELAYS はそれに nos.lol を追加。プロフィール/outbox取得用のlookupリレーは purplepag.es, index.hzrd149.com, indexer.coracle.social

## ソースコード

**ファイル**: `src/const.ts` (行 37-52)

```typescript
/** The recommended lookup relays used to fetch users profiles and outboxes when the user has no published lookup relay list */
export const RECOMMENDED_LOOKUP_RELAYS = relays([
  "wss://purplepag.es/",
  "wss://index.hzrd149.com",
  "wss://indexer.coracle.social",
]);

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
- `DEFAULT_FALLBACK_RELAYS` = relay.primal.net, relay.damus.io。outbox を公開していないユーザーのイベント取得に使うデフォルトリレー。
- `RECOMMENDED_FALLBACK_RELAYS` = 上記に nos.lol を追加。outbox を持たないユーザー向けの推奨リレー。
- `RECOMMENDED_LOOKUP_RELAYS`（lookup = プロフィール・outbox 取得用）= purplepag.es, index.hzrd149.com, indexer.coracle.social。
- いずれも `src/const.ts` にハードコードされている。
- 前回調査から lookup リレーに indexer.coracle.social が追加されている。

## 参考
- https://github.com/hzrd149/nostrudel/blob/master/src/const.ts

---
← [README](../README.md)
