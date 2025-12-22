← [README](../README.md)

# 野雨 (Nosame) リアクション取得方法

## 結論
- **リアクション取得方法**: リアクション取得なし（送信のみ）

## ソースコード

**ファイル**: `app.js` (行 178-189)

購読フィルター:
```javascript
_sendSubscription(ws) {
    if (ws.readyState !== WebSocket.OPEN) return;
    ws.send(JSON.stringify([
        "REQ",
        this.subId,
        {
            kinds: [NOSTR_KINDS.TEXT],  // kind:1 のみ
            limit: CONFIG.NOSTR_REQ_LIMIT,
            since: Math.floor(Date.now() / 1000) - CONFIG.NOSTR_REQ_SINCE_SECONDS_AGO
        }
    ]));
}
```

**ファイル**: `app.js` (行 234)

リアクション送信:
```javascript
kind: NOSTR_KINDS.REACTION,
```

## 説明

- 購読は `kind:1` (テキストノート) のみ
- `kind:7` (リアクション) の購読・取得機能なし
- リアクション送信機能のみ実装

## 参考
- https://github.com/invertedtriangle358/Nosame/blob/main/app.js

---
← [README](../README.md)
