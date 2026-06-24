← [README](../README.md)

# noStrudel リアクション取得方法

## 結論
- **リアクション取得方法**: イベントごとに `ReactionsModel` でローカル event store からリアクション(kind:7)を取得する方式。以前は applesauce-loaders の `reactionsLoader` で都度ロードしていたが、現行は `events.model(ReactionsModel, event)` でストアを購読する形に変更されている(`reactionsLoader` 自体は残るが UI からは直接呼ばれていない)。

## ソースコード

**ファイル**: `src/models/reactions.ts` (行 5-7)

```typescript
export function ReactionsQuery(event: NostrEvent, _relays?: string[]): Model<NostrEvent[]> {
  return (events) => events.model(ReactionsModel, event);
}
```

## 説明
- `useEventReactions`(`src/hooks/use-event-reactions.ts`)→ `ReactionsQuery` → applesauce-common の `ReactionsModel` という経路でリアクションを取得する。
- リアクション取得は applesauce ライブラリ側の event store / loader 機構に委譲され、ストアを購読する形でリアクション(kind:7)を集める。
- 第2引数 `relays` は現在未使用(`_relays`)。`extraRelays`=`fallbackRelays` は `src/services/loaders.ts:72-76` に定義されるが、現状 UI には配線されていない。

## 参考
- https://github.com/hzrd149/nostrudel/blob/master/src/models/reactions.ts

---
← [README](../README.md)
