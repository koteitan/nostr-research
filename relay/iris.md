# iris リレー取得方法

## 結論
- **リレー取得方法**: Outboxモデル (kind:10002 NIP-65)

## ソースコード

**ファイル**: `src/lib/ndk/outbox/tracker.ts`

```typescript
export class OutboxTracker extends EventEmitter {
  public data: LRUCache<Hexpubkey, OutboxItem>
  private ndk: NDK

  async trackUsers(items: NDKUser[] | Hexpubkey[], skipCache = false) {
    // ...
    getRelayListForUsers(pubkeys, this.ndk, skipCache, 1000, relayHints)
      .then((relayLists: Map<Hexpubkey, NDKRelayList>) => {
        for (const [pubkey, relayList] of relayLists) {
          let outboxItem = this.data.get(pubkey)!
          outboxItem ??= new OutboxItem("user")

          if (relayList) {
            outboxItem.readRelays = new Set(normalize(relayList.readRelayUrls))
            outboxItem.writeRelays = new Set(normalize(relayList.writeRelayUrls))
            // ...
          }
        }
      })
  }
}
```

**ファイル**: `src/lib/ndk/events/kinds/relay-list.ts`

```typescript
/**
 * Represents a relay list for a user, ideally coming from a NIP-65 kind:10002
 */
export class NDKRelayList extends NDKEvent {
  static kinds = [NDKKind.RelayList]

  get readRelayUrls(): WebSocket["url"][] {
    return this.tags
      .filter((tag) => tag[0] === "r" || tag[0] === "relay")
      .filter((tag) => !tag[2] || (tag[2] && tag[2] === READ_MARKER))
      .map((tag) => tryNormalizeRelayUrl(tag[1]))
      .filter((url) => !!url) as WebSocket["url"][]
  }

  get writeRelayUrls(): WebSocket["url"][] {
    return this.tags
      .filter((tag) => tag[0] === "r" || tag[0] === "relay")
      .filter((tag) => !tag[2] || (tag[2] && tag[2] === WRITE_MARKER))
      .map((tag) => tryNormalizeRelayUrl(tag[1]))
      .filter((url) => !!url) as WebSocket["url"][]
  }
}
```

## 説明

- OutboxTracker がユーザーの kind:10002 (NIP-65) を取得
- readRelays と writeRelays を分離して管理
- LRUCache でリレーリストをキャッシュ

## 参考
- https://github.com/irislib/iris-client/blob/main/src/lib/ndk/outbox/tracker.ts
- https://github.com/irislib/iris-client/blob/main/src/lib/ndk/events/kinds/relay-list.ts
