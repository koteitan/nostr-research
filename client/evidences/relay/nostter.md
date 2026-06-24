← [README](../README.md)

# nostter リレー取得方法

## 結論
- **リレー取得方法**: kind:10002 (NIP-65) を優先取得し rxNostr.setDefaultRelays に適用。なければ kind:3 (Contacts) の content から取得。まず localStorage キャッシュ、無ければ metadataRelays から購読。

## ソースコード

**ファイル**: `web/src/lib/author/RelayList.ts` (行 60-72)

```typescript
public static apply(eventsMap: Map<number, Event>) {
	const kind10002 = eventsMap.get(10002);
	const kind3 = eventsMap.get(3);
	if (kind10002 !== undefined && kind10002.tags.length > 0) {
		rxNostr.setDefaultRelays(kind10002.tags);
	} else if (kind3 !== undefined && kind3.content !== '') {
		rxNostr.setDefaultRelays(
			[...parseRelayJson(kind3.content)].map(([url, { read, write }]) => {
				return { url, read, write };
			})
		);
	}
}
```

## 説明
- kind:10002 (NIP-65) を最優先で利用し、tags をそのまま `rxNostr.setDefaultRelays` に渡してデフォルトリレーを設定する。
- kind:10002 が無い（または tags が空）場合は、フォールバックとして kind:3 (Contacts) の content を `parseRelayJson` で解析し、read/write 付きでデフォルトリレーに設定する。
- `fetchEvents` は kind `[3, 10002]` をまず localStorage キャッシュから取得し、無ければ metadataRelays から購読する流れになっている。
- 別ファイル `web/src/lib/RelayList.ts` はフォロイーの kind:10002 をまとめて取得する用途であり、本ファイルとは役割が異なる。

## 参考
- https://github.com/SnowCait/nostter/blob/main/web/src/lib/author/RelayList.ts

---
← [README](../README.md)
