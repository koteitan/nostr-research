← [README](../README.md)

# flowgazer Bootstrap Relay

## 結論
- **Bootstrap リレー**: 単一リレー固定: wss://r.kojira.io/（localStorage の relayUrl で上書き可）

## ソースコード

**ファイル**: `app.js` (行 276-280)

```javascript
const savedRelay = localStorage.getItem('relayUrl');
const defaultRelay = 'wss://r.kojira.io/';
const relay = savedRelay || defaultRelay;
await this.connectRelay(relay);
```

## 説明
- 起動時に1リレーへ接続する単一リレー構成。
- デフォルトは r.kojira.io、localStorage に relayUrl があればそれを優先。
- relay-manager.js は WebSocket 1本のみを管理。
- NIP-65/複数リレー非対応。
- lite/ 版は別途 nos.lol を既定にしている。

## 参考
- https://github.com/ompomz/flowgazer/blob/main/app.js

---
← [README](../README.md)
