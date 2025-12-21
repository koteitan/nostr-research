# flowgazer リアクション取得方法

## 結論
- **リアクション取得方法**: 自分の投稿へのリアクションを取得

## ソースコード

**ファイル**: `app.js` (行 135-165, 279-292)

Likes フィルタ (自分宛のリアクション):
```javascript
// === Likes フィルタ (自分宛のリアクション等) ===
if (myPubkey) {
  // kind:7 (リアクション)
  filters.push({
    kinds: [7],
    '#p': [myPubkey],
    limit: 50
  });
}
```

自分宛のリアクション取得:
```javascript
fetchReceivedLikes() {
  const myPubkey = window.nostrAuth.pubkey;
  if (!myPubkey) return;

  // kind:7 (リアクション)
  const filters = [{
    kinds: [7],
    '#p': [myPubkey],
    limit: 50
  }];
  // ...
}
```

## 説明

- `#p: [myPubkey]` で自分宛のリアクションをフィルタ
- 他の投稿のリアクション数は取得していない
- Likes タブで自分宛のリアクションを表示

## 参考
- https://github.com/ompomz/flowgazer/blob/main/app.js
