← [README](../README.md)

# iris Bootstrap Relay

## 結論
- **Bootstrap リレー**: ハードコードされたデフォルトリレー (PRODUCTION_RELAYS): temp.iris.to, vault.iris.to, relay.damus.io, relay.snort.social, relay.primal.net

## ソースコード

**ファイル**: `src/shared/constants/relays.ts` (行 3-28)

```typescript
const PRODUCTION_RELAYS = [
  "wss://temp.iris.to/",
  "wss://vault.iris.to/",
  "wss://relay.damus.io/",
  "wss://relay.snort.social/",
  "wss://relay.primal.net/",
]

function getDefaultRelays() {
  if (import.meta.env.VITE_USE_TEST_RELAY) {
    return TEST_RELAY
  }
  if (import.meta.env.VITE_USE_LOCAL_RELAY) {
    return LOCAL_RELAY
  }
  return PRODUCTION_RELAYS
}

export const DEFAULT_RELAYS = getDefaultRelays()
```

## 説明
- ユーザーのリレーリストを取得する前に、`PRODUCTION_RELAYS` の 5 つのリレー (temp.iris.to, vault.iris.to, relay.damus.io, relay.snort.social, relay.primal.net) に接続する。
- 前回調査から変更: nos.lol が削除され relay.primal.net が追加された。
- 環境変数 `VITE_USE_TEST_RELAY` / `VITE_USE_LOCAL_RELAY` により、テスト用 (temp.iris.to)・ローカル (ws://127.0.0.1:7777) のリレーに切替可能。
- 通常のプロダクションビルドでは `getDefaultRelays()` が `PRODUCTION_RELAYS` を返し、`DEFAULT_RELAYS` としてエクスポートされる。

## 参考
- https://github.com/irislib/iris-client/blob/main/src/shared/constants/relays.ts

---
← [README](../README.md)
