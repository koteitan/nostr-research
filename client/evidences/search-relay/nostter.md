← [README](../README.md)

# nostter 検索リレー

## 結論
- **検索リレー**: nostr.wine, search.nos.today
- 環境変数 `VITE_SEARCH_RELAYS` で上書き可能（カンマ区切り）
- `filter.search` 指定時や検索ページのキーワード検索時にこのリレーを使用

## ソースコード

**ファイル**: `web/src/lib/Constants.ts` (行 132-136)

```typescript
const searchRelayUrls = import.meta.env.VITE_SEARCH_RELAYS
	? import.meta.env.VITE_SEARCH_RELAYS.split(',')
	: ['wss://nostr.wine/', 'wss://search.nos.today/'];

export const searchRelays = searchRelayUrls.map((url) => url.trim());
```

## 説明
- デフォルトの検索リレーは `nostr.wine` と `search.nos.today` の 2 つ。
- 環境変数 `VITE_SEARCH_RELAYS` が設定されていればカンマ区切りで上書きでき、各 URL は trim される。
- `web/src/lib/Search.ts`（186行）では `filter.search` の有無で `searchRelays` か `readRelays` を切り替える。
- `web/src/lib/timelines/SearchTimeline.svelte.ts`（66行）でも検索リレーを参照する。
- `web/src/routes/(app)/search/+page.svelte`（124行）ではキーワードがある場合（`hasKeyword`）のみ検索リレーを使用する。
- 前回調査時の `relay.nostr.band` が削除され `nostr.wine` に置き換わった。

## 参考
- https://github.com/SnowCait/nostter/blob/main/web/src/lib/Constants.ts

---
← [README](../README.md)
