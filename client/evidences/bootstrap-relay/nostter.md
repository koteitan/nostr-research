← [README](../README.md)

# nostter Bootstrap Relay

## 結論
- **Bootstrap リレー**: ハードコードされたデフォルトリレー: nos.lol, relay.damus.io（環境変数 VITE_DEFAULT_RELAYS で上書き可能）。ロケールが ja の場合は日本リレー4件を追加。
- **日本語設定時追加**: relay-jp.nostr.wirednet.jp, yabu.me, r.kojira.io, nrelay-jp.c-stellar.net

## ソースコード

**ファイル**: `web/src/lib/Constants.ts` (行 91-124)

```typescript
const defaultRelayUrls = import.meta.env.VITE_DEFAULT_RELAYS
	? import.meta.env.VITE_DEFAULT_RELAYS.split(',')
	: ['wss://nos.lol/', 'wss://relay.damus.io/'];

export const defaultRelays = defaultRelayUrls.map((url) => ({
	url: url.trim(),
	read: true,
	write: true
}));

export const localizedRelays = {
	ja: [
		{ url: 'wss://relay-jp.nostr.wirednet.jp/', read: true, write: true },
		{ url: 'wss://yabu.me/', read: true, write: true },
		{ url: 'wss://r.kojira.io/', read: true, write: true },
		{ url: 'wss://nrelay-jp.c-stellar.net/', read: true, write: true }
	]
};
```

## 説明
- デフォルトリレーは `nos.lol` と `relay.damus.io` の2件をハードコード。
- 環境変数 `VITE_DEFAULT_RELAYS` が設定されていれば、カンマ区切りで上書き可能。
- ロケールが `ja` の場合のみ、`web/src/routes/+layout.ts` の `addDefaultRelays(localizedRelays.ja)` により日本リレー4件を追加する。
- 前回調査時にあった `relay.nostr.band` はデフォルトから削除され、現在は2件のみとなっている。

## 参考
- https://github.com/SnowCait/nostter/blob/main/web/src/lib/Constants.ts

---
← [README](../README.md)
