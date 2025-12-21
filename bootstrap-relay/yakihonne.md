# yakihonne Bootstrap Relay

## 結論
- **Bootstrap リレー**: nostr-01.yakihonne.com, nostr-02.yakihonne.com, relay.damus.io, relay.nostr.band, relay.nsec.app, monitorlizard.nostr1.com

## ソースコード

**ファイル**: `src/Content/Relays.js`

```javascript
const relaysOnPlatform = [
  "wss://nostr-01.yakihonne.com",
  "wss://nostr-02.yakihonne.com",
  "wss://relay.damus.io",
  "wss://relay.nostr.band",
  "wss://relay.nsec.app",
  "wss://monitorlizard.nostr1.com/",
];
```

**ファイル**: `src/Helpers/NDKInstance.js`

```javascript
import { relaysOnPlatform } from "@/Content/Relays";

const ndkInstance = new NDK({
  explicitRelayUrls: relaysOnPlatform,
  enableOutboxModel: true,
  // ...
});
```

## 説明

- relaysOnPlatform が初期接続リレー
- NDK の enableOutboxModel が有効
- yakihonne 自身のリレー (nostr-01, nostr-02) を含む

## 参考
- https://github.com/nicehashdev/yakihonne-web-app/blob/main/src/Content/Relays.js
- https://github.com/nicehashdev/yakihonne-web-app/blob/main/src/Helpers/NDKInstance.js
