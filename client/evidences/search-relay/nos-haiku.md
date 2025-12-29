← [README](../README.md)

# Nos Haiku 検索リレー

## 結論
- **検索リレー**: search.nos.today

## ソースコード

**ファイル**: `src/lib/config.ts` (行 26)

```typescript
export const searchRelays = ['wss://search.nos.today/'];
```

**ファイル**: `src/lib/resource.ts` (行 1270-1275)

```typescript
} else if (query !== undefined) {
    for (const relay of searchRelays) {
        relaySet.add(normalizeURL(relay));
    }
    const f: LazyFilter = { ...filterBase, kinds: [40, 41], search: query };
    filtersB.push(f);
}
```

## 説明

- `searchRelays` に `search.nos.today` のみ定義
- query パラメータが存在する場合、`search.nos.today` に `search` フィルタを送信
- NIP-50 検索対応

## 参考
- https://github.com/nikolat/nos-haiku/blob/main/src/lib/config.ts

---
← [README](../README.md)
