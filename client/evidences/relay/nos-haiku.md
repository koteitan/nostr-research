← [README](../README.md)

# Nos Haiku リレー取得方法

## 結論
- **リレー取得方法**: Outbox モデル (NIP-65 / kind:10002)。ユーザ自身の kind:10002 を setDefaultRelays に設定し、ホームタイムラインはフォロイーの write リレーを getReadRelaysWithOutboxModel / getOutboxes で抽出。kind:10002 が無い場合のみ defaultRelays にフォールバック

## ソースコード

**ファイル**: `src/lib/resource.ts` (行 1303-1346)

```typescript
if (f.authors.length >= 2) {
	const relays = getReadRelaysWithOutboxModel(
		f.authors,
		this.getReplaceableEvent,
		this.#relayFilter
	);
	for (const relayUrl of relays) {
		relaySet.add(relayUrl);
	}
} else {
	for (const pubkey of f.authors) {
		const event10002 = this.getReplaceableEvent(10002, pubkey);
		if (event10002 === undefined) continue;
		const relays = getOutboxes(event10002).filter(this.#relayFilter).slice(0, this.#limitRelay);
		for (const relayUrl of relays) relaySet.add(relayUrl);
	}
}
```

## 説明
- Outbox モデル (NIP-65 / kind:10002) を採用している。
- `setRelays()` がユーザの kind:10002 を `getRelaysToUseFromKind10002Event` でパースし、`rxNostr.setDefaultRelays` に設定する (resource.ts:700-708)。
- ホームタイムラインでは、フォロイーが複数いる場合 (`f.authors.length >= 2`) に `getReadRelaysWithOutboxModel` でフォロイーの write リレーを一括抽出する。
- フォロイーが 1 件の場合は、その pubkey の kind:10002 を `getOutboxes` でパースしてリレーを抽出する。
- kind:10002 が無い場合のみ `defaultRelays` にフォールバックする。
- applesauce-core / rx-nostr の `getOutboxes`・`getReadRelaysWithOutboxModel` を使用している。

## 参考
- https://github.com/nikolat/nos-haiku/blob/main/src/lib/resource.ts

---
← [README](../README.md)
