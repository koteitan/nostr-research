← [README](../README.md)

# ぬるぬる 検索リレー

## 結論
- **検索リレー**: search.nos.today (NIP-50)

## ソースコード

**ファイル**: `lib/nostr.js` (行 53,78, 1341-1367)

```javascript
const ENV_SEARCH_RELAY = process.env.NEXT_PUBLIC_SEARCH_RELAY
export const SEARCH_RELAY = ENV_SEARCH_RELAY || 'wss://search.nos.today'

export async function searchNotes(query, options = {}) {
  const filter = { kinds: [1], search: query, limit }
  const events = await p.querySync([SEARCH_RELAY], filter)
  ...
}
```

## 説明
- NIP-50 の全文検索を利用する。
- デフォルトの検索リレーは `wss://search.nos.today`。
- 環境変数 `NEXT_PUBLIC_SEARCH_RELAY` で上書き可能。
- `searchNotes`（kind:1）・`searchProfiles`（kind:0）の両方でこの単一の検索リレーを使用する。

## 参考
- https://github.com/tami1A84/null--nostr/blob/main/lib/nostr.js

---
← [README](../README.md)
