# ぬるぬる (nullnull) Bootstrap Relay

## 結論
- **Bootstrap リレー**: yabu.me (単一リレー)

## ソースコード

**ファイル**: `lib/nostr.js` (行 22)

```javascript
// Default relay for all operations
export const DEFAULT_RELAY = 'wss://yabu.me'
```

**ファイル**: `lib/nostr.js` (行 38-45)

```javascript
// Get single default relay from localStorage
export function getDefaultRelay() {
  if (typeof window !== 'undefined') {
    const saved = localStorage.getItem('defaultRelay')
    if (saved) return saved
  }
  return DEFAULT_RELAY
}
```

## 説明

- 単一リレーアーキテクチャ
- デフォルトは `yabu.me`
- `localStorage` の `defaultRelay` キーで変更可能

## 参考
- https://github.com/tami1A84/null--nostr/blob/main/lib/nostr.js
