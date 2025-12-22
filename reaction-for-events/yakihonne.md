← [README](../README.md)

# yakihonne リアクション取得方法

## 結論
- **リアクション取得方法**: イベントごとに #e タグで取得 (since で増分取得)

## ソースコード

**ファイル**: `src/Hooks/useNoteStats.js` (行 81-108)

```javascript
const response = await getSubData([
  {
    kinds: [7],
    "#e": [noteID],
    since: actions.likes.since,
  },
  {
    kinds: [6],
    "#e": [noteID],
    since: actions.reposts.since,
  },
  {
    kinds: [1],
    "#q": [noteID],
    since: actions.quotes.since,
  },
  {
    kinds: [1],
    "#e": [noteID],
    since: actions.replies.since,
  },
  {
    kinds: [9735],
    "#p": [notePubkey],
    "#e": [noteID],
    since: actions.zaps.since,
  },
]);
```

## 説明

- 各イベントごとに `#e` タグフィルタでリアクションを取得
- kind:7 (リアクション), kind:6 (リポスト), kind:1 (返信/引用), kind:9735 (Zap) を同時取得
- `since` パラメータで増分取得 (前回取得以降のデータのみ)
- IndexedDB (Dexie) にキャッシュして重複排除
- WOT (Web of Trust) スコアでフィルタリング

## 参考
- https://github.com/nicehashdev/yakihonne-web-app/blob/main/src/Hooks/useNoteStats.js

---
← [README](../README.md)
