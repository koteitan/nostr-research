# Lumilumi 検索リレー

## 結論
- **検索リレー**: directory.yabu.me, purplepag.es, relay.nostr.band, indexer.coracle.social

## ソースコード

**ファイル**: `src/lib/stores/relays.ts`

```typescript
export const relaySearchRelays = [
  // kind 0 (ユーザのプロフィール) と kind 10002 (利用中のリレーリスト) に特化
  "wss://directory.yabu.me",    // kind0, 3, 10002特化
  "wss://purplepag.es",         // https://purplepag.es/what
  "wss://relay.nostr.band",
  "wss://indexer.coracle.social",
];
```

## 説明

- `relaySearchRelays` がユーザ検索に使用される
- Kind 0（プロフィール）、Kind 3（コンタクト）、Kind 10002（リレーリスト）に特化したリレー

## 参考
- https://github.com/TsukemonoGit/lumilumi/blob/main/src/lib/stores/relays.ts
