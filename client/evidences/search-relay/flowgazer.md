← [README](../README.md)

# flowgazer 検索リレー

## 結論
- **検索リレー**: なし（NIP-50 全文検索は未実装）

## ソースコード

**ファイル**: `app.js` (行 29-90)

```javascript
buildStreamPhaseFilters(myPubkey) {
  // kinds:[1,6]/authors/#p/#e ベースのフィルタのみ。search フィールドは無し
}
```

## 説明
- `buildStreamPhaseFilters` は `kinds:[1,6]` に `authors` / `#p` / `#e` を組み合わせたフィルタのみを構築しており、`search` フィールド（NIP-50）を持たない。
- コード全体に NIP-50 の `search` フィルタや検索専用リレーへの言及はない。
- grep で見つかる `search` は `URLSearchParams`（`timeline.js:829`）のみで、Nostr の全文検索とは無関係。

## 参考
- https://github.com/ompomz/flowgazer/blob/main/app.js

---
← [README](../README.md)
