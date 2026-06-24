← [README](../README.md)

# Yakihonne Bootstrap Relay

## 結論
- **Bootstrap リレー**: ハードコードされた relaysOnPlatform 6リレーを NDK の explicitRelayUrls として初期接続 (enableOutboxModel: true)

## ソースコード

**ファイル**: `src/Content/Relays.js` (行 1-8)

```javascript
const relaysOnPlatform = [
  "wss://nostr-01.yakihonne.com",
  "wss://nostr-02.yakihonne.com",
  "wss://relay.damus.io",
  "wss://relay.nsec.app",
  "wss://nos.lol",
  "wss://monitorlizard.nostr1.com/",
];
```

## 説明
- `relaysOnPlatform` に6つのリレーURLがハードコードされている。
- `src/Helpers/NDKInstance.js` (行 6-14) で `explicitRelayUrls: relaysOnPlatform`, `enableOutboxModel: true` として NDK を初期化し、これらを初期接続先とする。
- yakihonne 自身のリレー (nostr-01 / nostr-02) を含む。
- リレー一覧が更新され `nos.lol` が追加、旧 `relay.nostr.band` は削除された。

## 参考
- https://github.com/YakiHonne/web-app/blob/main/src/Content/Relays.js

---
← [README](../README.md)
