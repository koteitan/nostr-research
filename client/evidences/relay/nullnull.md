← [README](../README.md)

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

## ハードコードされたリレーURL

| リレーURL | ファイル | 用途 |
|-----------|---------|------|
| `wss://yabu.me` | lib/nostr.js:60 | デフォルトリレー |
| `wss://search.nos.today` | lib/nostr.js:78 | 検索リレー (NIP-50) |
| `wss://relay-jp.nostr.wirednet.jp` | lib/nostr.js:69 | フォールバックリレー |
| `wss://r.kojira.io` | lib/nostr.js:70 | フォールバックリレー |
| `wss://relay.damus.io` | lib/nostr.js:71 | フォールバックリレー |
| `wss://relay.nsec.app` | lib/nip46.js:6 | NIP-46 リモート署名用 |
| `wss://yabu.me` | components/BadgeDisplay.js:132 | バッジ取得用 |
| `wss://relay.damus.io` | components/BadgeDisplay.js:132 | バッジ取得用 |
| `wss://nos.lol` | components/BadgeDisplay.js:132 | バッジ取得用 |
| `wss://yabu.me` | components/SchedulerApp.js:21 | カレンダーイベント用 |
| `wss://relay-jp.nostr.wirednet.jp` | components/SchedulerApp.js:22 | カレンダーイベント用 |

### 用途別まとめ

1. **デフォルト/Bootstrap**: `yabu.me`
2. **検索 (NIP-50)**: `search.nos.today`
3. **フォールバック**: `relay-jp.nostr.wirednet.jp`, `r.kojira.io`, `relay.damus.io`
4. **NIP-46 (リモート署名)**: `relay.nsec.app`
5. **バッジ取得 (NIP-58)**: `yabu.me`, `relay.damus.io`, `nos.lol`
6. **カレンダー (NIP-52)**: `yabu.me`, `relay-jp.nostr.wirednet.jp`

## 参考
- https://github.com/tami1A84/null--nostr/blob/main/lib/nostr.js

---
← [README](../README.md)
