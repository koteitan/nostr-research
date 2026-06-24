← [README](../README.md)

# flowgazer リアクション取得方法

## 結論
- **リアクション取得方法**: ストリームから機会的にカウント＋自分宛(#p/#e)を明示取得。任意ノートへの個別リアクション購読は無し

## ソースコード

**ファイル**: `data-store.js` (行 116-140)

```javascript
if (event.kind === 6 || event.kind === 7) {
  this._updateReactionCount(event);
}
// _updateReactionCount:
const targetId = event.tags.find(t => t[0] === 'e')?.[1];
const counts = this.reactionCounts.get(targetId);
if (event.kind === 6) counts.reposts++;
else if (event.kind === 7) counts.reactions++;
```

## 説明
- リアクション(kind:7)/リポスト(kind:6)はストリーム到着時に `dataStore._updateReactionCount` で eventId 別に集計し、バッジ表示する（`timeline.js` の `createReactionBadge`）。
- 明示取得は自分宛のみ: `buildStreamPhaseFilters` が `kinds:[7]`/`[6]` の `#p:[myPubkey]` と `kinds:[6,7]` の `#e:[自分の投稿ID]` を購読（`app.js:65-78`）。
- likes タブ用に `kinds:[1,6,7]` の `#p:[myPubkey]` を `received-notifications-init` で取得（`app.js:1077-1100`）。
- グローバルストリームは `kinds:[1,6]`（kind:7 はグローバルでは購読しない）ため、フォロー外ノートのリアクション数は網羅的には集計されない。
- 任意ノートへの個別リアクション購読の仕組みは無い。

## 参考
- https://github.com/ompomz/flowgazer/blob/main/data-store.js

---
← [README](../README.md)
