← [README](../README.md)

# Primal 検索リレー

## 結論
- **検索リレー**: Primalキャッシュサーバー経由 (専用検索リレーなし)

## ソースコード

検索機能はPrimalキャッシュサーバー経由で実行されます。

**ファイル**: `src/sockets.tsx` (行 76-82)

```typescript
cacheServer =
  localStorage.getItem('cacheServer') ||
  import.meta.env.PRIMAL_CACHE_URL;

let s = new WebSocket(cacheServer);
```

## 説明

- Primalは独自のキャッシュサーバーを使用
- 検索機能はキャッシュサーバーが提供
- 標準的なNostr検索リレー (relay.nostr.band等) は直接使用しない
- キャッシュサーバーが検索インデックスを管理

## 参考
- https://github.com/PrimalHQ/primal-web-app/blob/main/src/sockets.tsx

---
← [README](../README.md)
