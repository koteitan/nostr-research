← [README](../README.md)

# Nos Haiku Bootstrap Relay

## 結論
- **Bootstrap リレー**: indexerRelays (directory.yabu.me, purplepag.es, user.kindpag.es, indexer.coracle.social) で kind:10002 を取得。kind:10002 未取得時の汎用イベント取得には defaultRelays (nrelay.c-stellar.net, nostream.ocha.one, nostr.compile-error.net) を使用

## ソースコード

**ファイル**: `src/lib/config.ts` (行 14-26)

```typescript
export const defaultRelays: string[] = [
	'wss://nrelay.c-stellar.net/',
	'wss://nostream.ocha.one/',
	'wss://nostr.compile-error.net/'
];
export const indexerRelays = [
	'wss://directory.yabu.me/',
	'wss://purplepag.es/',
	'wss://user.kindpag.es/',
	'wss://indexer.coracle.social/'
];
```

## 説明
- `indexerRelays` でユーザーの kind:10002 (NIP-65 リレーリスト) を取得する。`fetchKind10002()` が `indexerRelays` を使って kind:10002 を取得 (resource.ts:1000)。
- `defaultRelays` は rxNostr の初期リレーとして設定され (resource.ts:158)、kind:10002 が取得できない場合の汎用イベント取得に使われる。
- プロフィール取得には `profileRelays` を使用する。

## 参考
- https://github.com/nikolat/nos-haiku/blob/main/src/lib/config.ts

---
← [README](../README.md)
