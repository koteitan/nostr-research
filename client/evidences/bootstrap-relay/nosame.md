← [README](../README.md)

# 野雨 Bootstrap Relay

## 結論
- **Bootstrap リレー**: ハードコードされた日本リレー4つ (relay-jp.nostr.wirednet.jp, yabu.me, r.kojira.io, nostr.compile-error.net) を初期接続先として使用

## ソースコード

**ファイル**: `config.js` (行 9-14)

```javascript
DEFAULT_RELAYS: [
    "wss://relay-jp.nostr.wirednet.jp",
    "wss://yabu.me",
    "wss://r.kojira.io",
    "wss://nostr.compile-error.net",
],
```

## 説明
- `DEFAULT_RELAYS` に日本リレー4つがハードコードされ、初期接続先として使用される。
- 旧 `app.js` が `config.js` 等に分割された。
- リレー一覧が変更され、`relay.barine.co` が削除、`nostr.compile-error.net` が追加された。
- ロケール分岐はなく、常に日本リレー固定。

## 参考
- https://github.com/invertedtriangle358/Nosame/blob/main/config.js

---
← [README](../README.md)
