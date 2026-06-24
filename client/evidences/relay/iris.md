← [README](../README.md)

# iris リレー取得方法

## 結論
- **リレー取得方法**: Outbox モデル (NIP-65 kind:10002)。NDK を enableOutboxModel で初期化し、OutboxTracker が各ユーザーの kind:10002 リレーリストを取得して read/write リレーを管理する。初期接続先はユーザー設定リレー（無ければ DEFAULT_RELAYS）。

## ソースコード

**ファイル**: `src/lib/ndk/outbox/tracker.ts` (行 73-138)

```typescript
async trackUsers(items: NDKUser[] | Hexpubkey[], skipCache = false) {
  ...
  getRelayListForUsers(pubkeys, this.ndk, skipCache, 1000, relayHints)
    .then((relayLists: Map<Hexpubkey, NDKRelayList>) => {
      for (const [pubkey, relayList] of relayLists) {
        let outboxItem = this.data.get(pubkey)!
        outboxItem ??= new OutboxItem("user")
        if (relayList) {
          outboxItem.readRelays = new Set(normalize(relayList.readRelayUrls))
          outboxItem.writeRelays = new Set(normalize(relayList.writeRelayUrls))
```

## 説明
- OutboxTracker が `getRelayListForUsers` で各ユーザーの kind:10002 (NIP-65) リレーリストを取得し、readRelays / writeRelays を分離管理する Outbox モデルを採用。
- NDK は `src/lib/ndk` にベンダリングされている（以前は `@nostr-dev-kit/ndk` 依存だったが、現在はリポジトリ内に取り込み済み）。
- 初期化は `src/utils/ndk.ts` の `initNDK` で `enableOutboxModel` / `explicitRelayUrls` を設定（`src/utils/relayRuntime.ts` の `resolveRelayRuntimeConfig` 経由、ユーザー設定 `store.ndkOutboxModel` に従う）。
- 初期接続先（explicitRelayUrls）はユーザー設定リレー、無ければ `DEFAULT_RELAYS`。
- NIP-65 の読み取りは `src/lib/ndk/events/kinds/relay-list.ts` が担当。

## 参考
- https://github.com/irislib/iris-client/blob/main/src/lib/ndk/outbox/tracker.ts

---
← [README](../README.md)
