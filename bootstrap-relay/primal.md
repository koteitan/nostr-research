# Primal Bootstrap Relay

## 結論
- **Bootstrap リレー**: Primalキャッシュサーバー経由で取得

## ソースコード

**ファイル**: `src/sockets.tsx` (行 76-82)

```typescript
export const connect = () => {
  if (isNotConnected()) {
    cacheServer =
      localStorage.getItem('cacheServer') ||
      import.meta.env.PRIMAL_CACHE_URL;

    let s = new WebSocket(cacheServer);
    // ...
  }
};
```

**ファイル**: `src/lib/relays.ts`

```typescript
export const getDefaultRelays = (subid: string) => {
  sendMessage(JSON.stringify([
    "REQ",
    subid,
    {cache: ["get_default_relays"]},
  ]))
};
```

**ファイル**: `src/lib/PrimalNip46.ts` (NIP-46 用)

```typescript
relays: ['wss://relay.primal.net'],
```

## 説明

- Primalキャッシュサーバー (PRIMAL_CACHE_URL) に接続
- `get_default_relays` でデフォルトリレー一覧を取得
- キャッシュサーバーがリレー接続を管理
- NIP-46 では `relay.primal.net` を使用

## 参考
- https://github.com/PrimalHQ/primal-web-app/blob/main/src/sockets.tsx
- https://github.com/PrimalHQ/primal-web-app/blob/main/src/lib/relays.ts
