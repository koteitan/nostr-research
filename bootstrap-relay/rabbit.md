# Rabbit Bootstrap Relay

## 結論
- **Bootstrap リレー**: relay.damus.io, nos.lol, relay.snort.social, relay.nostr.wirednet.jp
- **日本語設定時追加**: relay-jp.nostr.wirednet.jp, nostr.holybea.com, r.kojira.io, yabu.me

## ソースコード

**ファイル**: `src/core/relayUrls.ts`

```typescript
export const relaysGlobal: string[] = [
  'wss://relay.damus.io',
  'wss://nos.lol',
  'wss://relay.snort.social',
  'relay.nostr.wirednet.jp',
];

// 日本語タイムライン用のリレーリスト（日本国内限定・日本語中心のリレー）
export const relaysForJapaneseTL: string[] = [
  'wss://relay-jp.nostr.wirednet.jp',
  'wss://nostr.holybea.com',
  'wss://r.kojira.io',
  'wss://yabu.me',
];

export const relaysInJP: string[] = [...relaysForJapaneseTL];
```

**ファイル**: `src/core/useConfig.ts` (行 111-117)

```typescript
const initialRelays = (): string[] => {
  const relayUrls = [...relaysGlobal];
  if (window.navigator.language.includes('ja')) {
    relayUrls.push(...relaysInJP);
  }
  return relayUrls;
};
```

## 参考
- https://github.com/syusui-s/rabbit/blob/main/src/core/relayUrls.ts
