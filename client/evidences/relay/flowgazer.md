← [README](../README.md)

# flowgazer リレー取得方法

## 結論
- **リレー取得方法**: 設定→localStorage（単一リレー）。NIP-65/kind:10002 は未使用

## ソースコード

**ファイル**: `app.js` (行 986-994)

```javascript
async connectRelay(url) {
  try {
    document.getElementById('relay-url').value = url;
    await window.relayManager.connect(url);
    localStorage.setItem('relayUrl', url);
  } catch (err) { ... }
```

## 説明
- ホームタイムラインも接続中の単一リレーから取得する（`buildStreamPhaseFilters` で global/following/myposts などのフィルタを同一リレーに REQ）。
- ユーザーが UI でリレーURLを変更すると `localStorage.relayUrl` に保存して再接続する。
- アウトボックス/リレーリスト取得は行わない。

## 参考
- https://github.com/ompomz/flowgazer/blob/main/app.js

---
← [README](../README.md)
