# Amethyst 検索リレー

## 結論
- **検索リレー**: relay.nostr.band, nostr.wine, relay.noswhere.com, search.nos.today

## ソースコード

**ファイル**: `amethyst/src/main/java/com/vitorpamplona/amethyst/model/AccountSettings.kt` (行 81)

```kotlin
val DefaultSearchRelayList = setOf(Constants.band, Constants.wine, Constants.where, Constants.nostoday)
```

**ファイル**: `amethyst/src/main/java/com/vitorpamplona/amethyst/model/Constants.kt`

```kotlin
val band = RelayUrlNormalizer.normalize("wss://relay.nostr.band")
val wine = RelayUrlNormalizer.normalize("wss://nostr.wine")
val where = RelayUrlNormalizer.normalize("wss://relay.noswhere.com")
val nostoday = RelayUrlNormalizer.normalize("wss://search.nos.today")

val defaultSearchRelaySet = setOf(band, wine, where)
```

**ファイル**: `amethyst/src/main/java/com/vitorpamplona/amethyst/model/nip51Lists/searchRelays/SearchRelayListState.kt`

```kotlin
suspend fun normalizeSearchRelayListWithBackup(note: Note): Set<NormalizedRelayUrl> =
    searchListEvent(note)?.let { decryptionCache.relays(it) }?.ifEmpty { null }
        ?: DefaultSearchRelayList
```

## 説明

- DefaultSearchRelayList: 4つの検索リレー
- SearchRelayListEvent でユーザーの検索リレー設定を取得
- リレーリストがない場合は DefaultSearchRelayList にフォールバック
- NIP-50 検索対応

## 参考
- https://github.com/vitorpamplona/amethyst/blob/main/amethyst/src/main/java/com/vitorpamplona/amethyst/model/AccountSettings.kt
- https://github.com/vitorpamplona/amethyst/blob/main/amethyst/src/main/java/com/vitorpamplona/amethyst/model/nip51Lists/searchRelays/SearchRelayListState.kt
