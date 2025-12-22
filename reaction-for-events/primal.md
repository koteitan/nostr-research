← [README](../README.md)

# Primal リアクション取得方法

## 結論
- **リアクション取得方法**: Primalキャッシュサーバー経由 (NoteStats kind:10000100)

## ソースコード

**ファイル**: `src/lib/notes.tsx` (行 839-843)

```typescript
sendMessage(JSON.stringify([
  "REQ",
  subid,
  {cache: ["event_actions", { ...payload }]},
]));
```

**ファイル**: `src/lib/feed.ts` (行 325-329)

```typescript
sendMessage(JSON.stringify([
  "REQ",
  subid,
  {cache: ["thread_view", payload]},
]));
```

**ファイル**: `src/constants.ts` (行 136)

```typescript
export enum Kind  {
  // ...
  NoteStats = 10_000_100,
  NoteActions = 10_000_115,
  // ...
}
```

## 説明

- Primalはキャッシュサーバーがリアクションを事前集計
- `event_actions` でイベントごとのアクション取得
- `thread_view` でスレッド全体の情報を取得
- NoteStats (kind:10000100) で統計情報を返却
- 標準的な kind:7 の購読ではなく、集約済みデータを使用

## 参考
- https://github.com/PrimalHQ/primal-web-app/blob/main/src/lib/notes.tsx
- https://github.com/PrimalHQ/primal-web-app/blob/main/src/lib/feed.ts

---
← [README](../README.md)
