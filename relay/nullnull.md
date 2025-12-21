# ぬるぬる (nullnull) リレー取得方法

## 結論
- **リレー取得方法**: 設定→localStorage (単一リレー)

## ソースコード

**ファイル**: `lib/nostr.js` (行 38-45, 55-62)

```javascript
// Get single default relay from localStorage
export function getDefaultRelay() {
  if (typeof window !== 'undefined') {
    const saved = localStorage.getItem('defaultRelay')
    if (saved) return saved
  }
  return DEFAULT_RELAY
}

// Get relays array (always single relay for simplicity)
export function getReadRelays() {
  return [getDefaultRelay()]
}

export function getWriteRelays() {
  return [getDefaultRelay()]
}
```

## 説明

- 単一リレーアーキテクチャ
- kind:10002 (NIP-65) は使用していない
- `localStorage` の `defaultRelay` キーで設定を保存

## 参考
- https://github.com/tami1A84/null--nostr/blob/main/lib/nostr.js
