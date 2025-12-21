# Nos Haiku Bootstrap Relay

## 結論
- **Bootstrap リレー (indexerRelays)**: directory.yabu.me, purplepag.es, user.kindpag.es, indexer.coracle.social

## ソースコード

**ファイル**: `src/lib/config.ts`

```typescript
export const defaultRelays: string[] = [
  'wss://nrelay.c-stellar.net/',
  'wss://nostream.ocha.one/',
  'wss://nostr.compile-error.net/'
];

// kind:10002 を取得するためのインデクサーリレー
export const indexerRelays = [
  "wss://directory.yabu.me/",
  "wss://purplepag.es/",
  "wss://user.kindpag.es/",
  "wss://indexer.coracle.social/"
];

export const profileRelays = [
  'wss://directory.yabu.me/',
  'wss://user.kindpag.es/'
];

export const searchRelays = ['wss://search.nos.today/'];
```

## 説明

- `indexerRelays`: kind:10002（リレーリスト）を取得するためのリレー。これが Bootstrap リレーとして機能する
- `defaultRelays`: 一般的なイベント取得用
- `profileRelays`: プロフィール取得用
- `searchRelays`: NIP-50 検索用

## 参考
- https://github.com/nikolat/nos-haiku/blob/main/src/lib/config.ts
