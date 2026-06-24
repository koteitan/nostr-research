← [README](../README.md)

# 野雨 リレー取得方法

## 結論
- **リレー取得方法**: 設定→localStorage（NIP-65 不使用）。localStorage の relays キーから取得し、未保存時は DEFAULT_RELAYS にフォールバック

## ソースコード

**ファイル**: `nostr-core.js` (行 207-213)

```javascript
getRelays() {
    return this._load("relays", [...CONFIG.DEFAULT_RELAYS]);
}

saveRelays(relays) {
    this._save("relays", relays);
}
```

## 説明

- StorageManager クラス内で localStorage の `relays` キーからリレー一覧を取得する
- 値が未保存の場合は `CONFIG.DEFAULT_RELAYS` にフォールバックする
- kind:10002 (NIP-65) / outbox モデルは未使用
- ユーザーが設定画面で編集したリレーを localStorage に保存し、全リレーへ同一フィルタで購読する

## 参考
- https://github.com/invertedtriangle358/Nosame/blob/main/nostr-core.js

---
← [README](../README.md)
