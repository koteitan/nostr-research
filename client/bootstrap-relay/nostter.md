← [README](../README.md)

# nostter Bootstrap Relay

## 結論
- **Bootstrap リレー**: relay.nostr.band, nos.lol, relay.damus.io
- **日本語設定時追加**: relay-jp.nostr.wirednet.jp, yabu.me, r.kojira.io, nrelay-jp.c-stellar.net

## ソースコード

**ファイル**: `web/src/lib/Constants.ts` (行 87-128)

```typescript
export const defaultRelays = [
	{
		url: 'wss://relay.nostr.band/',
		read: true,
		write: true
	},
	{
		url: 'wss://nos.lol/',
		read: true,
		write: true
	},
	{
		url: 'wss://relay.damus.io/',
		read: true,
		write: true
	}
];

export const localizedRelays = {
	ja: [
		{
			url: 'wss://relay-jp.nostr.wirednet.jp/',
			read: true,
			write: true
		},
		{
			url: 'wss://yabu.me/',
			read: true,
			write: true
		},
		{
			url: 'wss://r.kojira.io/',
			read: true,
			write: true
		},
		{
			url: 'wss://nrelay-jp.c-stellar.net/',
			read: true,
			write: true
		}
	]
};
```

## 参考
- https://github.com/SnowCait/nostter/blob/main/web/src/lib/Constants.ts

---
← [README](../README.md)
