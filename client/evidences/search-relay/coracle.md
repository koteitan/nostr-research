← [README](../README.md)

# Coracle 検索リレー

## 結論
- **検索リレー**: 専用検索リレー nostr.wine, search.nos.today を使用（NIP-50）。Router.get().Search().getUrls() 経由で env.SEARCH_RELAYS に問い合わせ

## ソースコード

**ファイル**: `src/engine/requests.ts` (行 143-172)

```ts
myRequest({
  autoClose: true,
  skipCache: true,
  relays: Router.get().Search().getUrls(),
  filters: [{kinds: [0], search: term, limit: 100}],
  onEvent,
  ...
})
```

## 説明
- `Router.get().Search().getUrls()` で取得した専用検索リレーに問い合わせる。
- 対象リレーは `.env.template` の `VITE_SEARCH_RELAYS=nostr.wine,search.nos.today`（前回の relay.nostr.band は削除され 2 リレーに）。
- `createPeopleLoader` でプロフィール（kind:0）検索を行い、`search` パラメータ付きフィルタを使うため NIP-50 に準拠。

## 参考
- https://github.com/coracle-social/coracle/blob/master/src/engine/requests.ts

---
← [README](../README.md)
