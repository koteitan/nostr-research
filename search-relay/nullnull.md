# ぬるぬる (nullnull) 検索リレー

## 結論
- **検索リレー**: search.nos.today

## ソースコード

**ファイル**: `lib/nostr.js` (行 25)

```javascript
// Search relay (NIP-50)
export const SEARCH_RELAY = 'wss://search.nos.today'
```

**ファイル**: `lib/nostr.js` (行 1212-1238)

```javascript
// NIP-50: Search notes using search relay
export async function searchNotes(query, options = {}) {
  const { limit = 50, since, until, authors } = options
  const p = getPool()

  try {
    const filter = {
      kinds: [1],
      search: query,
      limit
    }

    const events = await p.querySync(
      [SEARCH_RELAY],
      filter
    )

    return events.sort((a, b) => b.created_at - a.created_at)
  } catch (e) {
    console.error('Search failed:', e)
    return []
  }
}
```

## 参考
- https://github.com/tami1A84/null--nostr/blob/main/lib/nostr.js
