← [README](../README.md)

# nostter リアクション取得方法

## 結論
- **リアクション取得方法**: ノート詳細ページ (/[slug=note]) を開いた時に kinds:[1,6,7,9735] / '#e':[eventId] でワンショット購読し、返信・リポスト・リアクション・Zap をまとめて取得する。

## ソースコード

**ファイル**: `web/src/routes/(app)/[slug=note]/+page.svelte` (行 201-208)

```typescript
const relatedEventsReq = createRxOneshotReq({
	filters: [
		{
			kinds: [1, 6, 7, 9735],
			'#e': [eventId]
		}
	]
});
```

## 説明
- `createRxOneshotReq` で関連イベントをワンショット購読する。
- フィルターは kinds:[1,6,7,9735] / '#e':[eventId] で、対象ノートに紐づくイベントをまとめて取得する。
- 取得結果は `filterByKind` で返信(1)/リポスト(6)/リアクション(7)/Zap(9735) に振り分ける。
- タイムライン上の個々のノートでは一括バッチ購読ではなく、詳細ページ表示時に取得する方式。

## 参考
- https://github.com/SnowCait/nostter/blob/main/web/src/routes/(app)/[slug=note]/+page.svelte

---
← [README](../README.md)
