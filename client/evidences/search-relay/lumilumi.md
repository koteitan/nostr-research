← [README](../README.md)

# Lumilumi 検索リレー

## 結論
- **検索リレー**: NIP-50 全文検索用リレー `nip50relays`: `search.nos.today`, `nostr.wine`, `cagliostr.compile-error.net`（`relay.nostr.band` は現在コメントアウト）。
- ユーザは kind:10007 (Search Relays List) でカスタム検索リレーを設定可能で、存在すればそちらを優先。

## ソースコード

**ファイル**: `src/lib/func/constants.ts` (行 49-56)

```ts
export const nip50relays = [
  //"wss://relay.nostr.band", //クソ長フィルターのとき（only foloweeのとき）nodataになる
  "wss://search.nos.today",
  // "wss://relay.noswhere.com",
  //"wss://bostr.nokotaro.com",
  "wss://nostr.wine",
  "wss://cagliostr.compile-error.net",
];
```

## 説明
- デフォルトの検索リレーは `nip50relays`（`search.nos.today`, `nostr.wine`, `cagliostr.compile-error.net`）。
- `relay.nostr.band` は「クソ長フィルターのとき nodata になる」ためコメントアウトされている。
- kind:10007 (Search Relays List) の取得・適用は `src/routes/search/+page.svelte` の `setSearchRelay`（101-141行）で行われる。
- ユーザの kind:10007 が存在すれば `toGlobalRelaySet` でデフォルトを上書きし、そちらを優先する。
- 旧調査からの変更点: `relay.nostr.band` を除外し、`nostr.wine` と `cagliostr.compile-error.net` を追加。

## 参考
- https://github.com/TsukemonoGit/lumilumi/blob/main/src/lib/func/constants.ts

---
← [README](../README.md)
