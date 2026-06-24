← [README](../README.md)

# Primal Bootstrap Relay

## 結論
- **Bootstrap リレー**: Primalキャッシュサーバーに最初に接続 (既定 `wss://cache2.primal.net/v1`)。優先リレー既定値は `wss://relay.primal.net`、NIP-46(リモート署名)では `wss://nrs.primal.net` を使用

## ソースコード

**ファイル**: `src/sockets.tsx` (行 74-93)

```tsx
export let cacheServer = '';

export const connect = () => {
  if (isNotConnected()) {
    cacheServer =
      localStorage.getItem('cacheServer') ||
      import.meta.env.PRIMAL_CACHE_URL;

    let s = new WebSocket(cacheServer);
```

## 説明
- 起動時はまずキャッシュサーバーへ WebSocket 接続する。接続先は `localStorage` の `cacheServer`、無ければ `.env` の `PRIMAL_CACHE_URL`（既定 `wss://cache2.primal.net/v1`）。
- 優先リレーの既定値は `.env` の `PRIMAL_PRIORITY_RELAYS="wss://relay.primal.net"`。優先リレーは `src/lib/relays.ts` の `getPreConfiguredRelays()` で取得する。
- NIP-46（リモート署名）利用時は `src/lib/PrimalNip46.ts`（行 50）で `wss://nrs.primal.net` を使用する。

## 参考
- https://github.com/PrimalHQ/primal-web-app/blob/main/src/sockets.tsx

---
← [README](../README.md)
