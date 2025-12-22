← [README](../README.md)

# flowgazer Bootstrap Relay

## 結論
- **Bootstrap リレー**: r.kojira.io

## ソースコード

**ファイル**: `app.js` (行 36-38)

```javascript
const savedRelay = localStorage.getItem('relayUrl');
const defaultRelay = 'wss://r.kojira.io/';
const relay = savedRelay || defaultRelay;
```

## 説明

- 単一リレーアーキテクチャ
- デフォルトは `r.kojira.io`
- `localStorage` の `relayUrl` キーで変更可能

## 参考
- https://github.com/ompomz/flowgazer/blob/main/app.js

---
← [README](../README.md)
