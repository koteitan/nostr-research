← [README](../README.md)

# Nos Haiku リレー取得方法

## 結論
- **リレー取得方法**: Outboxモデル (NIP-65)

## ソースコード

**ファイル**: `src/lib/resource.ts` (行 1304-1325)

```typescript
if (f.authors.length >= 2) {
    const relays = getReadRelaysWithOutboxModel(
        f.authors,
        this.getReplaceableEvent,
        this.#relayFilter
    );
    for (const relayUrl of relays) {
        relaySet.add(relayUrl);
    }
} else {
    for (const pubkey of f.authors) {
        const event10002: NostrEvent | undefined = this.getReplaceableEvent(10002, pubkey);
        if (event10002 === undefined) {
            continue;
        }
        const relays = getOutboxes(event10002)
            .filter(this.#relayFilter)
            .slice(0, this.#limitRelay);
        for (const relayUrl of relays) {
            relaySet.add(relayUrl);
        }
    }
}
```

**ファイル**: `src/lib/utils.ts` (行 746-764)

```typescript
export const getReadRelaysWithOutboxModel = (
    pubkeys: string[],
    getReplaceable: (kind: number, pubkey: string, d?: string) => NostrEvent | undefined,
    relayFilter: (relay: string) => boolean
): string[] => {
    // kind:10002 から Outbox リレーを抽出
    const event: NostrEvent | undefined = getReplaceable(10002, pubkey);
    const relays = getOutboxes(event).filter(relayFilter);
    // ...
}
```

## 説明

- ユーザの kind:10002 (NIP-65) を取得
- Outbox モデルでリレーを抽出（`getOutboxes()` 関数）
- フォローイーの Write リレーからイベントを取得

## 参考
- https://github.com/nikolat/nos-haiku/blob/main/src/lib/resource.ts

---
← [README](../README.md)
