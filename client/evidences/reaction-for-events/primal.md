← [README](../README.md)

# Primal リアクション取得方法

## 結論
- **リアクション取得方法**: Primalキャッシュサーバーがリアクションを事前集計。event_actions / thread_view で取得し、統計は NoteStats (kind:10000100) として返却 (kind:7 を直接購読しない)

## ソースコード

**ファイル**: `src/lib/notes.tsx` (行 1077-1115)

```typescript
export const getEventReactions = (eventId: string, kind: number, subid: string, offset = 0) => {
  ...
  let payload = { kind, limit: 20, offset };
  ...
  sendMessage(JSON.stringify([
    "REQ",
    subid,
    {cache: ["event_actions", { ...payload }]},
  ]));
};
```

## 説明

- Primalはキャッシュサーバーがリアクションを事前集計し、集約済みデータを使用
- `getEventReactions` が `event_actions` でイベントごとのアクションを取得
- スレッド全体は `src/lib/feed.ts` の `thread_view` / `multi_kind_thread_view` (行387, 365) で取得
- 統計情報は NoteStats (kind:10000100) として返却される
- `constants.ts` の定義: Reaction=7, NoteStats=10000100, NoteActions=10000115
- 標準的な kind:7 の購読は行わず、キャッシュサーバーの集約結果を利用

## 参考
- https://github.com/PrimalHQ/primal-web-app/blob/main/src/lib/notes.tsx

---
← [README](../README.md)
