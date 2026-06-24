← [README](../README.md)

# Amethyst リアクション取得方法

## 結論
- **リアクション取得方法**: イベント単位のバッチ取得 + リアルタイム購読。Compose の observeNoteReactions() で表示中ノートに対し EventFinderFilterAssemblerSubscription が REQ を購読。filterRepliesAndReactionsToNotes() がリレーごとにノートID を 'e' タグで集約し、RepliesAndReactionsKinds (ReactionEvent kind:7, RepostEvent kind:6, GenericRepostEvent 16, ReportEvent, LnZap 等) を limit=1000 でバッチ取得。

## ソースコード

**ファイル**: `amethyst/src/main/java/com/vitorpamplona/amethyst/service/relayClient/reqCommand/event/watchers/FilterRepliesAndReactionsToNotes.kt` (行 49-106)

```kotlin
val RepliesAndReactionsKinds =
    listOf(
        TextNoteEvent.KIND,
        ReactionEvent.KIND,
        RepostEvent.KIND,
        GenericRepostEvent.KIND,
        ReportEvent.KIND,
        LnZapEvent.KIND,
        ...
    )

fun filterRepliesAndReactionsToNotes(events: List<Note>, since: SincePerRelayMap?): List<RelayBasedFilter>? {
    val perRelayEventIds = mapOfSet {
        events.forEach { note ->
            note.relayUrlsForReactions().forEach { relay -> add(relay, note.idHex) }
        }
    }
    return perRelayEventIds.flatMap {
        ... Filter(kinds = RepliesAndReactionsKinds, tags = mapOf("e" to sortedList), since = since, limit = 1000)
```

## 説明
- 観測側は EventObservers.kt の observeNoteReactions()/observeNoteReactionCount() (218-253行) で、note.relayUrlsForReactions() のリレーへ購読する。
- 表示中ノートをまとめてリレー別に 'e' タグでバッチ REQ を発行する。
- since (EOSE) で差分取得し、limit=1000 を上限とする。
- LocalCache.consume(ReactionEvent) で Note.addReaction() を呼び、Flow を invalidate して UI を更新する。
- RepliesAndReactionsKinds は ReactionEvent (kind:7)、RepostEvent (kind:6)、GenericRepostEvent (16)、ReportEvent、LnZapEvent 等を含む。

## 参考
- https://github.com/vitorpamplona/amethyst/blob/main/amethyst/src/main/java/com/vitorpamplona/amethyst/service/relayClient/reqCommand/event/watchers/FilterRepliesAndReactionsToNotes.kt

---
← [README](../README.md)
