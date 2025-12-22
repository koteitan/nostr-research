← [README](../README.md)

# flowgazer リレー取得方法

## 結論
- **リレー取得方法**: 設定→localStorage (単一リレー)

## ソースコード

**ファイル**: `app.js` (行 36-38, 69)

```javascript
const savedRelay = localStorage.getItem('relayUrl');
const defaultRelay = 'wss://r.kojira.io/';
const relay = savedRelay || defaultRelay;

// URL保存
localStorage.setItem('relayUrl', url);
```

## 説明

- 単一リレーアーキテクチャ
- kind:10002 (NIP-65) は使用していない
- `localStorage` の `relayUrl` キーで設定を保存

## 参考
- https://github.com/ompomz/flowgazer/blob/main/app.js

---
← [README](../README.md)
