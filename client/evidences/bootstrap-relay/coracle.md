← [README](../README.md)

# Coracle Bootstrap Relay

## 結論
- **Bootstrap リレー**: 環境変数で定義したハードコードリレー。DEFAULT_RELAYS=relay.damus.io, nos.lol、INDEXER_RELAYS=relay.damus.io, purplepag.es, indexer.coracle.social を初期接続する

## ソースコード

**ファイル**: `src/engine/state.ts` (行 148-168, 809-814)

```typescript
export const env = {
  ...
  DEFAULT_RELAYS: fromCsv(import.meta.env.VITE_DEFAULT_RELAYS).map(normalizeRelayUrl) as string[],
  INDEXER_RELAYS: fromCsv(import.meta.env.VITE_INDEXER_RELAYS).map(normalizeRelayUrl) as string[],
  ...
}

const initialRelays = [
  ...env.DEFAULT_RELAYS,
  ...env.DVM_RELAYS,
  ...env.INDEXER_RELAYS,
  ...env.SEARCH_RELAYS,
]
...
ready.then(() => Promise.all(initialRelays.map(url => loadRelay(url))))
```

## 説明
- リレーURLは `.env.template` で環境変数として定義される。
- `DEFAULT_RELAYS=relay.damus.io,nos.lol`、`INDEXER_RELAYS=relay.damus.io,purplepag.es,indexer.coracle.social`。
- 前回調査と比べ `relay.nostr.band` は INDEXER から外れ、DEFAULT に `relay.damus.io` が追加された。
- 初期化時に DEFAULT / DVM / INDEXER / SEARCH の全リレーへ接続する。

## 参考
- https://github.com/coracle-social/coracle/blob/master/src/engine/state.ts

---
← [README](../README.md)
