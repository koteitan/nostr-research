← [README](../README.md)

# Yakihonne リアクション取得方法

## 結論
- **リアクション取得方法**: ノートごとに #e タグでサブスクリプション取得 (kind 7/6/1/9735 を since で増分取得)

## ソースコード

**ファイル**: `src/Hooks/useNoteStats.js` (行 81-108)

```javascript
const response = await getSubData([
  { kinds: [7], "#e": [noteID], since: actions.likes.since },
  { kinds: [6], "#e": [noteID], since: actions.reposts.since },
  { kinds: [1], "#q": [noteID], since: actions.quotes.since },
  { kinds: [1], "#e": [noteID], since: actions.replies.since },
  { kinds: [9735], "#p": [notePubkey], "#e": [noteID], since: actions.zaps.since },
]);
```

## 説明
- ノート単位で kind:7(リアクション)/6(リポスト)/1(返信・引用)/9735(Zap) を一括サブスクリプションで取得する。
- `since` を指定して前回取得以降のデータのみを増分取得する。
- 取得結果は IndexedDB(Dexie) にキャッシュして再利用する。
- WOT(Web of Trust)スコアで結果をフィルタリングする (`filterStatsByWot`)。

## 参考
- https://github.com/YakiHonne/web-app/blob/main/src/Hooks/useNoteStats.js

---
← [README](../README.md)
