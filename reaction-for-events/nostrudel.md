# noStrudel リアクション取得方法

## 結論
- **リアクション取得方法**: applesauce-loaders でイベントごとに取得

## ソースコード

**ファイル**: `src/hooks/use-event-reactions.ts`

```typescript
import { ReactionsQuery } from "../models/reactions";

export default function useEventReactions(event?: NostrEvent, relays?: string[]) {
  return useEventModel(ReactionsQuery, event ? [event, relays] : undefined);
}
```

**ファイル**: `src/models/reactions.ts`

```typescript
import { reactionsLoader } from "../services/loaders";

export function ReactionsQuery(event: NostrEvent, relays?: string[]): Model<NostrEvent[]> {
  return (events) =>
    defer(() => reactionsLoader(event, relays)).pipe(
      ignoreElements(),
      mergeWith(events.reactions(event))
    );
}
```

**ファイル**: `src/services/loaders.ts` (行 66-70)

```typescript
export const reactionsLoader = createReactionsLoader(pool, {
  cacheRequest,
  eventStore,
  extraRelays: localSettings.fallbackRelays,
});
```

## 説明

- applesauce-loaders の createReactionsLoader を使用
- 各イベントごとにリアクションを取得 (#e タグフィルタ)
- extraRelays として fallbackRelays を追加
- eventStore にキャッシュ

## 参考
- https://github.com/hzrd149/nostrudel/blob/main/src/hooks/use-event-reactions.ts
- https://github.com/hzrd149/nostrudel/blob/main/src/services/loaders.ts
