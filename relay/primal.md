# Primal リレー取得方法

## 結論
- **リレー取得方法**: Primalキャッシュサーバー経由 (kind:10002 も対応)

## ソースコード

**ファイル**: `src/sockets.tsx`

```typescript
cacheServer =
  localStorage.getItem('cacheServer') ||
  import.meta.env.PRIMAL_CACHE_URL;

let s = new WebSocket(cacheServer);
```

**ファイル**: `src/lib/relays.ts`

```typescript
export const getPreConfiguredRelays = () => {
  const rels: string[] = import.meta.env.PRIMAL_PRIORITY_RELAYS?.split(',') || [];

  return rels.reduce(
    (acc: NostrRelays, r: string) =>
      ({ ...acc, [r]: { read: true, write: true } }),
    {},
  );
};

export const getDefaultRelays = (subid: string) => {
  sendMessage(JSON.stringify([
    "REQ",
    subid,
    {cache: ["get_default_relays"]},
  ]))
};
```

## 説明

- Primalは独自のキャッシュサーバーを使用
- キャッシュサーバーがリレー接続を一元管理
- `PRIMAL_PRIORITY_RELAYS` 環境変数で追加リレーを設定可能
- kind:10002 (RelayList) もサポート (constants.ts で定義)

## 参考
- https://github.com/PrimalHQ/primal-web-app/blob/main/src/sockets.tsx
- https://github.com/PrimalHQ/primal-web-app/blob/main/src/lib/relays.ts
