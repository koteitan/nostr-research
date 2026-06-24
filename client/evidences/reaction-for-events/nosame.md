← [README](../README.md)

# 野雨 リアクション取得方法

## 結論
- **リアクション取得方法**: リアクション取得なし（送信のみ）。購読は kind:1 (テキスト) と kind:0 (メタデータ) のみで、kind:7 は送信専用

## ソースコード

**ファイル**: `nostr-core.js` (行 594-614)

```javascript
async sendReaction(target) {
    ...
    const event = {
        kind: NOSTR_KINDS.REACTION,
        content: "+",
        created_at: Math.floor(Date.now() / 1000),
        tags: [["e", target.id], ["p", target.pubkey]],
        pubkey,
    };
    const signed = await window.nostr.signEvent(event);
    this._broadcast(signed);
}
```

## 説明
- `_sendTextSubscription` / `_sendProfileSubscription` / `_sendProfileNotesSubscription` の購読は kind:1・kind:0 のみ。
- kind:7 のリアクションを購読・集計するコードは存在しない。
- `sendReaction` では kind:7 イベントを生成・署名し `_broadcast` で送信するのみ（受信・集計は行わない）。

## 参考
- https://github.com/invertedtriangle358/Nosame/blob/main/nostr-core.js

---
← [README](../README.md)
