← [README](../README.md)

# noStrudel 検索リレー

## 結論
- **検索リレー**: NIP-50検索。デフォルト検索リレーは relay.nostr.band, search.nos.today, relay.noswhere.com, filter.nostr.wine。ユーザーが検索リレーリスト(kind:10007系)を公開していればそれを優先

## ソースコード

**ファイル**: `src/const.ts` (行 24-29)

```typescript
export const DEFAULT_SEARCH_RELAYS = relays([
  "wss://relay.nostr.band",
  "wss://search.nos.today",
  "wss://relay.noswhere.com",
  "wss://filter.nostr.wine",
]);
```

## 説明
- NIP-50 に対応した検索を行う。
- `DEFAULT_SEARCH_RELAYS` に relay.nostr.band, search.nos.today, relay.noswhere.com, filter.nostr.wine の4つがデフォルトで定義されている。
- `src/hooks/use-search-relays.ts` で、ユーザーの検索リレーリスト(kind:10007系)があれば `getRelaysFromList` で取得し、無ければ `DEFAULT_SEARCH_RELAYS` を使用する。
- ローカルリレーが NIP-50 に対応していれば、そちらも検索に利用できる。

## 参考
- https://github.com/hzrd149/nostrudel/blob/master/src/const.ts

---
← [README](../README.md)
