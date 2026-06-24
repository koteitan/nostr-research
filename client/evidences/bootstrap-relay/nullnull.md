← [README](../README.md)

# ぬるぬる Bootstrap Relay

## 結論
- **Bootstrap リレー**: yabu.me (単一リレー、環境変数で上書き可)

## ソースコード

**ファイル**: `lib/nostr.js` (行 52-60, 115-121)

```javascript
const ENV_DEFAULT_RELAY = process.env.NEXT_PUBLIC_DEFAULT_RELAY
export const DEFAULT_RELAY = ENV_DEFAULT_RELAY || 'wss://yabu.me'

export function getDefaultRelay() {
  if (typeof window !== 'undefined') {
    const saved = localStorage.getItem('defaultRelay')
    if (saved) return saved
  }
  return DEFAULT_RELAY
}
```

## 説明
- デフォルトの接続先は `wss://yabu.me` の単一リレー。
- 環境変数 `NEXT_PUBLIC_DEFAULT_RELAY` が設定されていればそれで上書きされる。
- 未設定時はブラウザの `localStorage` の `defaultRelay` キーで変更可能。
- 複数リレーではなく単一リレーアーキテクチャを採用している。

## 参考
- https://github.com/tami1A84/null--nostr/blob/main/lib/nostr.js

---
← [README](../README.md)
