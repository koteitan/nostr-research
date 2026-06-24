← [README](../README.md)

# Yakihonne リレー取得方法

## 結論
- **リレー取得方法**: Outboxモデル (NIP-65 kind:10002)。ユーザの kind:10002 リレーリストを取得し、無ければ relaysOnPlatform にフォールバック

## ソースコード

**ファイル**: `src/Components/AppInit.js` (行 592-598)

```js
{
  kinds: [10002],
  authors: [userKeys.pub],
  since: lastRelaysTimestamp
    ? lastRelaysTimestamp + 1
    : lastRelaysTimestamp,
},
```

## 説明
- ユーザの kind:10002 (NIP-65 リレーリスト) を `authors: [userKeys.pub]` で購読して取得する。
- AppInit.js 行242-258で取得した kind:10002 から read/write リレーを抽出し `setUserRelays` に設定する。
- リレーリストが空の場合は `relaysOnPlatform` にフォールバックする。
- NDK の `enableOutboxModel` と `useOutboxRelays.js` により、フォロー先の kind:10002 から `outboxRelays` を pubkey 数順に構築する。

## 参考
- https://github.com/YakiHonne/web-app/blob/main/src/Components/AppInit.js

---
← [README](../README.md)
