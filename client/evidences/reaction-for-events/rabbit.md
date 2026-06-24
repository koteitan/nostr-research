← [README](../README.md)

# Rabbit リアクション取得方法

## 結論
- **リアクション取得方法**: バッチ取得 (2秒間隔、最大150件)。#e タグで kind:7 (Reaction) をフィルタし、設定リレー (config().relayUrls) へ subscribeMany

## ソースコード

**ファイル**: `src/nostr/useBatchedEvents.ts` (行 165-170, 300-316)

```typescript
const reactionsTasks = createTasks<ReactionsTask>({
  keyExtractor: (req) => req.mentionedEventId,
  filtersBuilder: (ids) => [{ kinds: [Kind.Reaction], '#e': ids }],
  // Use the last event id for compatibility
  eventKeyExtractor: (ev) => genericEvent(ev).lastTaggedEventId(),
});
...
return useBatch<BatchedEventsTask>(() => ({
  interval: 2000,
  batchSize: 150,
  executor: (tasks) => {
    ...
    const sub = pool().subscribeMany(config().relayUrls, filters, { ... });
```

## 説明
- `#e` タグに対象イベントIDを指定し、kind:7 (Reaction) をフィルタして取得する。
- 複数のリクエストをまとめるバッチ機構で処理し、2秒間隔・最大150件単位で購読する。
- 購読先は `config().relayUrls`、すなわちユーザー設定リレーである。
- Repost (kind:6) / Zap (kind:9735) も同じバッチ機構でまとめて取得している。

## 参考
- https://github.com/syusui-s/rabbit/blob/main/src/nostr/useBatchedEvents.ts

---
← [README](../README.md)
