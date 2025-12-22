← [README](../README.md)

# Coracle Bootstrap Relay

## 結論
- **Bootstrap リレー**: relay.damus.io, nos.lol (DEFAULT_RELAYS)
- **Indexer リレー**: relay.nostr.band, purplepag.es, indexer.coracle.social

## ソースコード

**ファイル**: `.env.template`

```bash
VITE_DEFAULT_RELAYS=wss://relay.damus.io,wss://nos.lol
VITE_INDEXER_RELAYS=wss://relay.nostr.band,wss://purplepag.es,wss://indexer.coracle.social
```

**ファイル**: `src/engine/state.ts` (行 139-159)

```typescript
export const env = {
  // ...
  DEFAULT_RELAYS: fromCsv(import.meta.env.VITE_DEFAULT_RELAYS).map(normalizeRelayUrl) as string[],
  INDEXER_RELAYS: fromCsv(import.meta.env.VITE_INDEXER_RELAYS).map(normalizeRelayUrl) as string[],
  // ...
}
```

**ファイル**: `src/engine/state.ts` (行 738-743, 757-760)

```typescript
const initialRelays = [
  ...env.DEFAULT_RELAYS,
  ...env.DVM_RELAYS,
  ...env.INDEXER_RELAYS,
  ...env.SEARCH_RELAYS,
]

// Configure router
routerContext.getDefaultRelays = always(env.DEFAULT_RELAYS)
routerContext.getIndexerRelays = always(env.INDEXER_RELAYS)
routerContext.getSearchRelays = always(env.SEARCH_RELAYS)
```

## 説明

- DEFAULT_RELAYS: 初期接続・フォールバック用
- INDEXER_RELAYS: kind:10002 (NIP-65) のリレーリスト取得用
- @welshman/router がリレー選択を管理
- 初期化時に全リレーに接続

## 参考
- https://github.com/coracle-social/coracle/blob/master/.env.template
- https://github.com/coracle-social/coracle/blob/master/src/engine/state.ts

---
← [README](../README.md)
