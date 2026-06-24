← [README](../README.md)

# Lumilumi Bootstrap Relay

## 結論
- **Bootstrap リレー**: Bootstrap リレー (relaySearchRelays): directory.yabu.me, purplepag.es, indexer.coracle.social, nostr.wine。kind:0/3/10002 の取得に特化したリレーで、最初に接続してユーザのリレーリスト(NIP-65)を探す。kind:10002 が見つからない場合のフォールバックが defaultRelays: nos.lol, nostr.bitcoiner.social, nostr-pub.wellorder.net, relay.snort.social。

## ソースコード

**ファイル**: `src/lib/stores/relays.ts` (行 1-28)

```typescript
export const relaySearchRelays = [
  //kind 0 (ユーザのプロフィール) と kind 10002 (利用中のリレーリスト) 特化
  "wss://directory.yabu.me", //kind0, 3, 10002特化
  "wss://purplepag.es", //https://purplepag.es/what
  "wss://indexer.coracle.social",
  "wss://nostr.wine",
  //"wss://relay.nostr.band",
];

export const defaultRelays = [
  "wss://nos.lol",
  "wss://nostr.bitcoiner.social",
  "wss://nostr-pub.wellorder.net/",
  "wss://relay.snort.social/",
];
```

## 説明
- `relaySearchRelays` は kind:0/3/10002 の取得に特化したブートストラップ用リレーで、最初に接続してユーザのリレーリスト (NIP-65) を探す。
- 構成は directory.yabu.me, purplepag.es, indexer.coracle.social, nostr.wine の 4 つ。
- kind:10002 が見つからない場合は `defaultRelays` (nos.lol, nostr.bitcoiner.social, nostr-pub.wellorder.net, relay.snort.social) にフォールバックする。
- 旧調査からリスト変更あり: `relaySearchRelays` に nostr.wine を追加・relay.nostr.band を除外。`defaultRelays` は nos.lol/nostr.bitcoiner.social/nostr-pub.wellorder.net/relay.snort.social に変更。

## 参考
- https://github.com/TsukemonoGit/lumilumi/blob/main/src/lib/stores/relays.ts

---
← [README](../README.md)
