← [README](../README.md)

# noStrudel リレー取得方法

## 結論
- **リレー取得方法**: Outboxモデル (kind:10002 NIP-65)

## ソースコード

**ファイル**: `src/services/outbox-subscriptions.ts`

```typescript
class OutboxSubscriptionsService {
  subscription(list: LoadableAddressPointer, filter: Filter): Observable<never> {
    return outboxCacheService.getOutboxMap(list).pipe(
      switchMap((outboxMap) => {
        const relays = Object.keys(outboxMap);
        if (relays.length === 0) {
          return NEVER;
        }

        const { authors, ...filterWithoutAuthors } = filter;
        return pool.outboxSubscription(outboxMap, filterWithoutAuthors).pipe(
          onlyEvents(),
          mapEventsToStore(eventStore),
          ignoreElements(),
        );
      }),
    );
  }
}
```

**ファイル**: `src/hooks/use-user-mailboxes.ts`

```typescript
export function useUserOutbox(pubkey?: string | ProfilePointer): string[] | undefined {
  const fallbacks = useObservableEagerState(localSettings.fallbackRelays);
  const unhealthy = useObservableEagerState(liveness.unhealthy$);
  const mailboxes = useUserMailboxes(pubkey);

  return useMemo(() => {
    if (!mailboxes) return fallbacks.filter((relay) => !unhealthy.includes(relay));
    return mailboxes.outboxes.filter((relay) => !unhealthy.includes(relay));
  }, [mailboxes, unhealthy, fallbacks]);
}
```

## 説明

- MailboxesModel (NIP-65) を使用してユーザーの inbox/outbox を取得
- outboxMap に基づいてリレーに購読
- 不健全なリレーはフィルタリング
- mailboxes がない場合は fallbackRelays を使用

## 参考
- https://github.com/hzrd149/nostrudel/blob/main/src/services/outbox-subscriptions.ts
- https://github.com/hzrd149/nostrudel/blob/main/src/hooks/use-user-mailboxes.ts

---
← [README](../README.md)
