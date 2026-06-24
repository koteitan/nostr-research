← [README](../README.md)

# Rabbit Bootstrap Relay

## 結論
- **Bootstrap リレー**: ハードコードされたグローバルリレー (relay.damus.io, nos.lol, relay.snort.social, relay.nostr.wirednet.jp)、ブラウザ言語が 'ja' の場合は日本語リレーを追加
- 日本語設定時追加: relay-jp.nostr.wirednet.jp, r.kojira.io, yabu.me

## ソースコード

**ファイル**: `src/core/relayUrls.ts` (行 1-15)

```typescript
export const relaysGlobal: string[] = [
  'wss://relay.damus.io',
  'wss://nos.lol',
  'wss://relay.snort.social',
  'relay.nostr.wirednet.jp',
];

// 日本語タイムライン用のリレーリスト
export const relaysForJapaneseTL: string[] = [
  'wss://relay-jp.nostr.wirednet.jp',
  'wss://r.kojira.io',
  'wss://yabu.me',
];

export const relaysInJP: string[] = [...relaysForJapaneseTL];
```

## 説明
- `relaysGlobal` にグローバルリレーがハードコードされている。
- `relaysForJapaneseTL` / `relaysInJP` に日本語タイムライン用のリレーが定義されている。
- リレー選択の判定は `src/core/useConfig.ts:129-135` の `initialRelays()` で行われ、`window.navigator.language.includes('ja')` が真の場合に日本語リレーを追加する。
- 旧版にあった nostr.holybea.com は削除された。

## 参考
- https://github.com/syusui-s/rabbit/blob/main/src/core/relayUrls.ts

---
← [README](../README.md)
