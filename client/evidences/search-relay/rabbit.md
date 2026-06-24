← [README](../README.md)

# Rabbit 検索リレー

## 結論
- **検索リレー**: relaysForSearching: relay.nostr.band, search.nos.today

## ソースコード

**ファイル**: `src/core/relayUrls.ts` (行 17)

```typescript
export const relaysForSearching: string[] = ['wss://relay.nostr.band', 'wss://search.nos.today'];
```

## 説明
- `relaysForSearching` に検索用リレー `wss://relay.nostr.band` と `wss://search.nos.today` を定義している。
- `src/components/column/SearchColumn.tsx:108` で `relayUrls: relaysForSearching` を指定し、kind:1 を search クエリ付きで購読する (NIP-50)。

## 参考
- https://github.com/syusui-s/rabbit/blob/main/src/core/relayUrls.ts

---
← [README](../README.md)
