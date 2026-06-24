← [README](../README.md)

# Yakihonne 検索リレー

## 結論
- **検索リレー**: 専用検索リレー searchRelays 3件 (search.nos.today, relay.ditto.pub, nostr.polyserv.xyz) + ユーザの kind:10007 リレー。NIP-50 search + #t タグ検索

## ソースコード

**ファイル**: `src/Content/Relays.js` (行 18-22)

```javascript
const searchRelays = [
  "wss://search.nos.today",
  "wss://relay.ditto.pub",
  "wss://nostr.polyserv.xyz",
];
```

## 説明
- `getSearchNdkInstance` (src/Helpers/SSGNDKInstance.js 行32-35) が `[...searchRelays, ...extRelays]` で専用 NDK インスタンスを生成する。
- `Search.js` (行242-247) で `getDataForSearch` を呼び、#t タグの各種大小文字バリエーション + search パラメータで検索する。
- 旧コードの relay.nostr.band は削除済み。
- ユーザは kind:10007 イベントで検索リレーをカスタマイズ可能。

## 参考
- https://github.com/YakiHonne/web-app/blob/main/src/Content/Relays.js

---
← [README](../README.md)
