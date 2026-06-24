← [README](../README.md)

# ぬるぬる リレー取得方法

## 結論
- **リレー取得方法**: 設定→localStorage (単一リレー、NIP-65不使用)

## ソースコード

**ファイル**: `lib/nostr.js` (行 137-147, 809-815)

```javascript
export function getReadRelays() {
  return [getDefaultRelay()]
}

export function getWriteRelays() {
  return [getDefaultRelay()]
}

// fetchEvents内: プライマリ + FALLBACK_RELAYS で取得
const allRelays = [primaryRelay, ...FALLBACK_RELAYS.filter(r => r !== primaryRelay)]
```

## 説明
- kind:10002 (NIP-65) / outbox は不使用。
- ホームタイムラインは `getReadRelays()` = `[getDefaultRelay()]` の単一リレーから取得。
- `fetchEvents` 内では `FALLBACK_RELAYS` (relay-jp.nostr.wirednet.jp, r.kojira.io, relay.damus.io) を追加して取得。
- リレーは localStorage の `defaultRelay` キーで保存・変更される。

## 参考
- https://github.com/tami1A84/null--nostr/blob/main/lib/nostr.js

---
← [README](../README.md)
