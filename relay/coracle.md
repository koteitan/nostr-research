# Coracle リレー取得方法

## 結論
- **リレー取得方法**: Outboxモデル (kind:10002 NIP-65) + @welshman/router

## ソースコード

**ファイル**: `.env.template`

```bash
VITE_INDEXER_RELAYS=wss://relay.nostr.band,wss://purplepag.es,wss://indexer.coracle.social
```

**ファイル**: `src/engine/state.ts`

```typescript
export const env = {
  // ...
  INDEXER_RELAYS: fromCsv(import.meta.env.VITE_INDEXER_RELAYS).map(normalizeRelayUrl) as string[],
  // ...
}

// Configure router
routerContext.getDefaultRelays = always(env.DEFAULT_RELAYS)
routerContext.getIndexerRelays = always(env.INDEXER_RELAYS)
routerContext.getSearchRelays = always(env.SEARCH_RELAYS)
routerContext.getLimit = () => getSetting("relay_limit")
```

**ファイル**: `src/engine/state.ts` (行 86-87)

```typescript
import {RELAYS, INBOX_RELAYS} from "@welshman/util"

export const metaKinds = [PROFILE, FOLLOWS, MUTES, RELAYS, INBOX_RELAYS]
```

## 説明

- @welshman/router がリレー選択を自動化
- INDEXER_RELAYS でユーザーのリレーリスト (kind:10002) を取得
- RELAYS = kind:10002 (NIP-65)
- INBOX_RELAYS = kind:10050 (NIP-17)
- relay_limit 設定でリレー数を制限 (デフォルト3)

## 参考
- https://github.com/coracle-social/coracle/blob/master/src/engine/state.ts
