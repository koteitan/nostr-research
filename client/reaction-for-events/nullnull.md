← [README](../README.md)

# ぬるぬる (nullnull) リアクション取得方法

## 結論
- **リアクション取得方法**: バッチ取得 (limit: 500)

## ソースコード

**ファイル**: `components/TimelineTab.js` (行 399)

```javascript
fetchEvents({ kinds: [7], '#e': eventIds, limit: 500 }, readRelays),
```

**ファイル**: `components/HomeTab.js` (行 407)

```javascript
{ kinds: [7], '#e': allPostIds, limit: 500 },
```

## 説明

- 複数の投稿IDをまとめて `#e` タグでフィルタ
- `limit: 500` で取得
- バッチ処理でリアクションを取得

## 参考
- https://github.com/tami1A84/null--nostr/blob/main/components/TimelineTab.js

---
← [README](../README.md)
