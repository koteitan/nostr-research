← [README](../README.md)

# nostter リレー取得方法

## 結論
- **リレー取得方法**: kind:10002 (NIP-65)

## ソースコード

**ファイル**: `web/src/lib/RelayList.ts`

```typescript
export class RelayList {
	static async fetchEvents(pubkeys: pubkey[]): Promise<Map<pubkey, Event>> {
		const eventsMap = new Map<pubkey, Event>();
		if (pubkeys.length === 0) {
			return eventsMap;
		}

		return new Promise((resolve) => {
			const req = createRxBackwardReq();
			rxNostr
				.use(req)
				.pipe(
					tie,
					uniq(),
					latestEach(({ event }) => event.pubkey)
				)
				.subscribe({
					next: (packet) => {
						console.debug('[rx-nostr kind 10002 next]', pubkeys, packet);
						eventsMap.set(packet.event.pubkey, packet.event);
					},
					complete: () => {
						console.debug('[rx-nostr kind 10002 complete]', pubkeys);
						resolve(eventsMap);
					}
				});
			req.emit([{ kinds: [10002], authors: pubkeys }]);
			req.over();
		});
	}
}
```

## 参考
- https://github.com/SnowCait/nostter/blob/main/web/src/lib/RelayList.ts

---
← [README](../README.md)
