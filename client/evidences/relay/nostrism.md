← [README](../README.md)

# Nostrism リレー取得方法

## 結論
- **リレー取得方法**: Outboxモデル（kind:10002 NIP-65）。ユーザー自身の kind:10002 をリレー設定として使い（初回は言語別デフォルトをシード）、Settings で編集して kind:10002 として署名・公開する。
- フォロー中カラムは自分の read リレーから購読（kind:1/6/16, authors=自分のフォロー＋自分, limit 100）。
- 著者カラム（著者数 ≤ 3、プロフィール等）は著者の kind:10002 の write リレーからも追加購読する（NIP-65 outbox）。

## ソースコード

**ファイル**: `composeApp/src/commonMain/kotlin/app/nostrdeck/data/EventRepository.kt` (行 798-832)

```kotlin
when {
    // [#8] 検索カラムは NIP-50 対応リレーへ
    !filter.search.isNullOrBlank() -> subscribeTargeted(columnId, SEARCH_RELAYS.toSet(), ...)
    filter.relays.isNotEmpty() -> subscribeTargeted(columnId, filter.relays.toSet(), proto)
    // [#209] プロフィール/指定npub（少数著者）は著者の書き込みリレー(NIP-65)からも取得（アウトボックス）。
    filter.authors.isNotEmpty() && filter.authors.size <= 3 -> subscribeAuthorOutbox(columnId, filter, proto)
    else -> subscribeAll(columnId, proto)
}

private fun subscribeAuthorOutbox(columnId: String, filter: ReqFilter, proto: Filter) {
    subscribeAll(columnId, proto)   // 自分のリレーで即購読
    scope.launch {
        // 著者の NIP-65 を indexer + 自分のリレーから取得。
        subscribeTargeted(relSub, INDEXER_RELAYS.toSet(), Filter(kinds = listOf(10002), authors = filter.authors, ...))
        subscribeAll(relSub, Filter(kinds = listOf(10002), authors = filter.authors, ...))
        delay(4000); unsubscribeAll(relSub)
        // DB から write リレーを取り出し、著者の投稿を outbox リレーからも購読（未接続のものだけ）。
        val writeRelays = filter.authors.flatMap { authorWriteRelays(it) }...
        if (writeRelays.isNotEmpty() && columnId in openColumns)
            subscribeTargeted("$columnId~outbox", writeRelays.toSet(), proto)
    }
}
```

## 説明
- リレー設定は DB（Inbox/Outbox + source）で管理し、Settings 画面で read/write を切り替えられる。現在のリレー設定は kind:10002（NIP-65）として署名・配信する（`publishSigned(UnsignedEvent(kind = 10002, ...))`）。初回は `defaultRelaysFor(language)` の言語別デフォルトをシードする（[bootstrap 参照](../bootstrap-relay/nostrism.md)）。
- **フォロー中カラム** (`subscribeFollowing`): 自分の kind:3 由来のフォロー集合を authors にして、自分の接続リレーから kind:1/6/16 を購読する（`authors = フォロー + 自分`, limit 100）。ここでは著者ごとの outbox は張らない。
- **著者カラム** (`subscribeAuthorOutbox`): 著者が 3 名以下のとき、まず自分のリレーで即購読し、並行して indexer + 自分のリレーから著者の kind:10002 を取得（4秒待ち）、その write リレー（marker 無し=両用 / "write"）のうち未接続のものを追加購読する。中間抜けの取りこぼしを減らす典型的なアウトボックス実装。追加接続は `HINT_RELAY_CAP = 16` で上限を設ける。
- write 専用（Outbox）リレーは常時購読せず、投稿の配信時に一時接続して EVENT を送る（NIP-65 outbox の書き込み側）。
- フォローがまだ無い新規アカウントには、フォロー中ユーザーの NIP-65 を集計する `fetchRelayRecommendations` によるレコメンドが主役。

## 参考
- https://github.com/ShinoharaTa/nostr-andloid-native-client/blob/main/composeApp/src/commonMain/kotlin/app/nostrdeck/data/EventRepository.kt

---
← [README](../README.md)
