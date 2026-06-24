← [README](../README.md)

# Damus 検索リレー

## 結論
- **検索リレー**: NIP-50検索リレーは使用しない。フルテキスト検索はローカルの nostrdb (ndb_text_search) で実行される。専用の検索リレーは存在しない

## ソースコード

**ファイル**: `nostrdb/Ndb.swift` (行 373-381)

```swift
func text_search(query: String, limit: Int = 128, order: NdbSearchOrder = .newest_first) throws -> [NoteKey] {
    return try withNdb({
        guard let txn = NdbTxn(ndb: self) else { return [] }
        var results = ndb_text_search_results()
        let res = query.withCString { q in
            let order = order == .newest_first ? NDB_ORDER_DESCENDING : NDB_ORDER_ASCENDING
            var config = ndb_text_search_config(order: order, limit: Int32(limit))
            return ndb_text_search(&txn.txn, q, &results, &config)
        }
        ...
    })
}
```

## 説明
- 検索リレーへの問い合わせは行わず、ローカルDB (nostrdb) に対して全文検索を実行する。
- `SearchResultsView.swift:145` と `PullDownSearch.swift:22` が `damus_state.ndb.text_search()` を呼び出す。
- ユーザー検索は `ndb.search_profile()` (`Profiles.swift:95`) を使用する。
- `SearchModel.swift` もリレー指定なしの `reader.query()` を使うのみ。

## 参考
- https://github.com/damus-io/damus/blob/master/nostrdb/Ndb.swift

---
← [README](../README.md)
