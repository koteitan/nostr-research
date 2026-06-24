← [README](../README.md)

# Nos Haiku リアクション取得方法

## 結論
- **リアクション取得方法**: バッチ取得 (rx-nostr backward req + bufferTime 1秒 + batch でフィルタマージ、limit 500) と前方リアルタイム購読の併用。kind:7 を #e(通常イベント) または #a(リプレース可能/アドレス可能イベント) で取得し、対象作成者の inbox(NIP-65) リレーを優先

## ソースコード

**ファイル**: `src/lib/resource.ts` (行 749-773)

```typescript
#fetchReaction = (event: NostrEvent) => {
	let filter: LazyFilter;
	const until = unixNow();
	if (isReplaceableKind(event.kind) || isAddressableKind(event.kind)) {
		const ap = { ...event, identifier: isAddressableKind(event.kind) ? (getTagValue(event, 'd') ?? '') : '' };
		filter = { kinds: [7], '#a': [getReplaceableAddressFromPointer(ap)], limit: this.#limitReaction, until };
	} else {
		filter = { kinds: [7], '#e': [event.id], limit: this.#limitReaction, until };
	}
	let options;
	const event10002 = this.getReplaceableEvent(10002, event.pubkey);
	if (event10002 !== undefined) {
		const relays = getInboxes(event10002).filter(this.#relayFilter).slice(0, this.#limitRelay);
		options = relays.length > 0 ? { relays } : undefined;
	}
	this.#rxReqB7.emit(filter, options);
};
```

## 説明
- イベント表示時に `#fetchReaction()` が呼ばれ、kind:7 (リアクション) を後方 (backward) req で取得する。
- 対象がリプレース可能/アドレス可能イベントなら `#a` タグ、通常イベントなら `#e` タグでフィルタする。
- バッチ化: `#secBufferTime=1000ms`、`#limitReaction=500` (resource.ts:123-124)。`batchedReq7 = rxReqB7.pipe(bufferTime, batch)` (resource.ts:213) により 1 秒分のリクエストをまとめてフィルタをマージしてから発行する。
- 取得先リレーは対象イベント作成者の kind:10002 (NIP-65) の inbox リレーを優先し、`#limitRelay` 件まで使用する。
- 前方 (forward) リアルタイム購読も併用 (resource.ts:1474-1492)。ページ種別ごとに `#p`/`#e`/`#a`/`#k` タグの `since` 付き kind:7 を継続購読する。

## 参考
- https://github.com/nikolat/nos-haiku/blob/main/src/lib/resource.ts

---
← [README](../README.md)
