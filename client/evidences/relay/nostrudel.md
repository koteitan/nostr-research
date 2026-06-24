← [README](../README.md)

# noStrudel リレー取得方法

## 結論
- **リレー取得方法**: Outboxモデル (NIP-65 kind:10002)。ホームタイムラインは対象ユーザーの outbox マップに対して購読する。outbox が取得できない場合は `localSettings.fallbackRelays` を使用

## ソースコード

**ファイル**: `src/services/outbox-subscriptions.ts` (行 21-47)

```typescript
subscription(list: LoadableAddressPointer, filter: Filter): Observable<never> {
  return outboxCacheService.getOutboxMap(list).pipe(
    switchMap((outboxMap) => {
      const relays = Object.keys(outboxMap);
      if (relays.length === 0) return NEVER;
      const { authors, ...filterWithoutAuthors } = filter;
      return pool.outboxSubscription(outboxMap, filterWithoutAuthors).pipe(
        onlyEvents(),
        mapEventsToStore(eventStore),
        ignoreElements(),
      );
    }),
  );
}
```

## 説明
- 各ユーザーの NIP-65 (kind:10002) アウトボックスを `outboxCacheService.getOutboxMap` で取得し、その outbox マップに対して `pool.outboxSubscription` で購読する。
- ホーム画面 (`src/views/home/index.tsx`) が `useOutboxTimelineLoader` と `outboxSubscriptionsService.subscription` を組み合わせて利用する。
- リレー選定の補助は `src/hooks/use-user-mailboxes.ts` の `useUserOutbox` が担い、mailboxes が無ければ `fallbackRelays` をフォールバックとして使う。
- タイムラインローダーは applesauce-loaders の `createOutboxTimelineLoader` を利用している。

## 参考
- https://github.com/hzrd149/nostrudel/blob/master/src/services/outbox-subscriptions.ts

---
← [README](../README.md)
