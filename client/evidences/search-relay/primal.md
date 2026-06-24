← [README](../README.md)

# Primal 検索リレー

## 結論
- **検索リレー**: 専用の全文検索リレーなし。検索はすべてPrimalキャッシュサーバー経由 (cache: "search" / "user_search" / "advanced_search")
- relay.nostr.band 等の外部検索リレーは直接使用しない

## ソースコード

**ファイル**: `src/lib/search.ts` (行 121-159)

```typescript
export const searchContent = (user_pubkey: string | undefined, subid: string, query: string, limit = 100) => {
  let payload: SearchPayload = { query: cleanQuery(query), limit };
  ...
  sendMessage(JSON.stringify([
    "REQ",
    subid,
    {cache: ["search", payload]},
  ]));
}
```

## 説明
- コンテンツ検索は `cache: ["search", payload]` を REQ で送信し、Primalキャッシュサーバーが処理する。
- ユーザー検索は `user_search` (行 117)、高度検索は `advanced_search` / `advanced_feed` (行 137, 157) を利用する。
- いずれもPrimalキャッシュサーバー経由であり、relay.nostr.band などの外部全文検索リレーには直接接続しない。

## 参考
- https://github.com/PrimalHQ/primal-web-app/blob/main/src/lib/search.ts

---
← [README](../README.md)
