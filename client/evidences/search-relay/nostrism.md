← [README](../README.md)

# Nostrism 検索リレー

## 結論
- **検索リレー**: `relay.nostr.band`, `relay.noswhere.sh`, `search.nos.today` の3件（NIP-50）。検索カラムは接続中リレーではなくこの専用リレー群へ問い合わせる（接続中リレーが NIP-50 未対応でも結果が出るように）。

## ソースコード

**ファイル**: `composeApp/src/commonMain/kotlin/app/nostrdeck/data/EventRepository.kt` (行 3753-3759, 801, 850-853)

```kotlin
val SEARCH_RELAYS = listOf(
    "wss://relay.nostr.band",
    "wss://relay.noswhere.sh",
    "wss://search.nos.today",
)
// [#210] 検索の取得上限。ローカル LIKE 表示（feedBySearch, LIMIT 300）に十分載るよう多めに取る。
const val SEARCH_FETCH_LIMIT = 300

// 購読分岐（行 801）:
!filter.search.isNullOrBlank() ->
    subscribeTargeted(columnId, SEARCH_RELAYS.toSet(), filter.toProtocol(limit = SEARCH_FETCH_LIMIT))

// NIP-50 フィルタ生成（行 850-853）: 単語=1語1フィルタ、タグ=#t
words.forEach { add(Filter(kinds = listOf(1), search = it, limit = limit)) }
if (hashtags.isNotEmpty()) add(Filter(kinds = listOf(1), hashtags = hashtags, limit = limit))
```

## 説明
- 検索カラムのフィルタに `search` が入っていると、`subscribeTargeted` で `SEARCH_RELAYS` の3件へだけ REQ を送る。到達性のばらつきに備えて複数へ投げ、どれか通れば結果が出るようにしている。
- キーワードは NIP-50（1語=1フィルタ、`kind:1`）、ハッシュタグは `#t`（複数値=OR）で分けて送る。取得上限は `SEARCH_FETCH_LIMIT = 300`。
- 受信結果は DB に取り込み、表示は `feedBySearch`（ローカル LIKE, LIMIT 300）で行う。
- なお本アプリの README ステータス表では検索が「未実装（バックログ）」と記されているが、コード上は上記の通り NIP-50 検索が実装済み（本調査はコードを正とする）。

## 参考
- https://github.com/ShinoharaTa/nostr-andloid-native-client/blob/main/composeApp/src/commonMain/kotlin/app/nostrdeck/data/EventRepository.kt

---
← [README](../README.md)
