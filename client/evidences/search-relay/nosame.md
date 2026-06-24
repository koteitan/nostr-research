← [README](../README.md)

# 野雨 検索リレー

## 結論
- **検索リレー**: なし（全文検索機能が未実装、NIP-50 非対応）

## ソースコード

**ファイル**: `nostr-core.js` (行 416-428)

```javascript
_sendTextSubscription(ws) {
    ws.send(JSON.stringify([
        "REQ",
        this.subId,
        {
            kinds: [NOSTR_KINDS.TEXT],
            limit: CONFIG.NOSTR_REQ_LIMIT,
            ...
        },
    ]));
}
```

## 説明
- コード全体に検索用の `REQ` や NIP-50 の `search` フィルタは存在しない。
- `_sendTextSubscription` は `kinds` と `limit` のみを指定しており、検索条件を持たない。
- `'search'` のヒットは `window.location.search`（URL クエリ処理）のみで Nostr 検索とは無関係。

## 参考
- https://github.com/invertedtriangle358/Nosame/blob/main/nostr-core.js

---
← [README](../README.md)
