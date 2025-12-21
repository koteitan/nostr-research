# 野雨 (Nosame) リレー取得方法

## 結論
- **リレー取得方法**: 設定→localStorage

## ソースコード

**ファイル**: `app.js` (行 59-68)

```javascript
class StorageManager {
    // ...
    _load(key, fallback) {
        return JSON.parse(localStorage.getItem(key)) || fallback;
    }

    _save(key, value) {
        localStorage.setItem(key, JSON.stringify(value));
    }

    getRelays() { return this._load("relays", [...CONFIG.DEFAULT_RELAYS]); }
    saveRelays(relays) { this._save("relays", relays); }
}
```

## 説明

- `localStorage` の `relays` キーに保存
- kind:10002 (NIP-65) は使用していない
- 保存されていない場合は DEFAULT_RELAYS にフォールバック

## 参考
- https://github.com/invertedtriangle358/Nosame/blob/main/app.js
