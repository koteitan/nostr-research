# Rabbit リアクション取得方法

## 結論
- **リアクション取得方法**: バッチ取得 (2秒間隔、最大150件)

## ソースコード

**ファイル**: `src/nostr/useBatchedEvents.ts` (行 165-170, 300-303)

```typescript
const reactionsTasks = createTasks<ReactionsTask>({
  keyExtractor: (req) => req.mentionedEventId,
  filtersBuilder: (ids) => [{ kinds: [Kind.Reaction], '#e': ids }],
  // Use the last event id for compatibility
  eventKeyExtractor: (ev) => genericEvent(ev).lastTaggedEventId(),
});
```

```typescript
return useBatch<BatchedEventsTask>(() => ({
  interval: 2000,
  batchSize: 150,
  executor: (tasks) => {
    // ...
  },
}));
```

## 説明

- 2秒間隔でバッチ取得
- 最大150件のタスクをまとめて処理
- `#e` タグで投稿IDを指定してkind:7（Reaction）をフィルタ

## 参考
- https://github.com/syusui-s/rabbit/blob/main/src/nostr/useBatchedEvents.ts
