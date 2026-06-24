← [README](../README.md)

# iris 検索リレー

## 結論
- **検索リレー**: 専用の検索リレーは無し。投稿検索は接続中のリレーへ NIP-50 search フィルタ + #t ハッシュタグフィルタを送信(buildSearchSubscriptionFilters)。プロフィール検索はローカル(Fuse.js)。

## ソースコード

**ファイル**: `src/shared/hooks/buildSearchSubscriptionFilters.ts` (行 75-100)

```typescript
const filterArray: NDKFilter[] = []

if (regularWords.length > 0) {
  // Plain recent notes fallback for relays without NIP-50 or word indexing.
  filterArray.push(boundedBaseFilter)
}
if (hashtags.length > 0) {
  filterArray.push({...boundedBaseFilter, "#t": buildHashtagVariants(filters.search, hashtags)})
}
if (regularWords.length > 0) {
  filterArray.push({...boundedBaseFilter, "#t": regularWords})
  filterArray.push({...boundedBaseFilter, search: regularWords.join(" ")})
}
return dedupeFilters(filterArray)
```

## 説明
- 専用の検索リレーは持たず、接続中のリレーへ検索フィルタを送信する。
- 検索ロジックは旧 `useFeedEvents.ts` から `buildSearchSubscriptionFilters.ts` に分離された。
- NIP-50 非対応リレー向けに、通常の最近の投稿フォールバックと `#t` フィルタも併用する。
- 取得結果はクライアント側で再フィルタする(`useFeedEvents.ts` 173-)。
- プロフィール検索は `src/workers/profile-search.ts` の Fuse.js(Dexie/IndexedDB キャッシュ対象)で行う。

## 参考
- https://github.com/irislib/iris-client/blob/main/src/shared/hooks/buildSearchSubscriptionFilters.ts

---
← [README](../README.md)
