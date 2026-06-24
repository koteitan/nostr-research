← [README](../README.md)

# ぬるぬる リアクション取得方法

## 結論
- **リアクション取得方法**: バッチ取得 (kind:7 を #e でまとめて limit:500)

## ソースコード

**ファイル**: `components/TimelineTab.js` (行 506-512)

```javascript
// Fetch reactions
if (pubkey && allPosts.length > 0) {
  const eventIds = allPosts.map(p => p.id)
  const [reactionEvents, myRepostEvents] = await Promise.all([
    fetchEvents({ kinds: [7], '#e': eventIds, limit: 500 }, readRelays),
    fetchEvents({ kinds: [6], authors: [pubkey], limit: 100 }, readRelays)
  ])
```

## 説明
- タイムライン読込後、表示中の投稿IDをまとめて `#e` タグでフィルタし、kind:7 を limit:500 で一括取得(バッチ)。
- フィルタ定義は `lib/filters.js` の `createReactionFilter` (kinds:[7], #e, limit=500)。
- `HomeTab.js`(行487)でも同様に kind:7 を #e/limit:500 で取得。
- リポスト(kind:6)も `Promise.all` で併せて取得。

## 参考
- https://github.com/tami1A84/null--nostr/blob/main/components/TimelineTab.js

---
← [README](../README.md)
