# Amethyst リレー取得方法

## 結論
- **リレー取得方法**: Outboxモデル (kind:10002 NIP-65) + IndexerRelayList

## ソースコード

**ファイル**: `amethyst/src/main/java/com/vitorpamplona/amethyst/model/AccountSettings.kt` (行 83)

```kotlin
val DefaultIndexerRelayList = setOf(Constants.purplepages, Constants.coracle, Constants.userkinds)
```

**ファイル**: `amethyst/src/main/java/com/vitorpamplona/amethyst/model/Constants.kt`

```kotlin
val purplepages = RelayUrlNormalizer.normalize("wss://purplepag.es")
val coracle = RelayUrlNormalizer.normalize("wss://indexer.coracle.social")
val userkinds = RelayUrlNormalizer.normalize("wss://user.kindpag.es")
```

**ファイル**: `amethyst/src/main/java/com/vitorpamplona/amethyst/model/nip51Lists/indexerRelays/IndexerRelayListState.kt`

```kotlin
suspend fun normalizeIndexerRelayListWithBackup(note: Note): Set<NormalizedRelayUrl> =
    indexListEvent(note)?.let { decryptionCache.relays(it) }?.ifEmpty { null }
        ?: DefaultIndexerRelayList
```

## 説明

- DefaultIndexerRelayList: purplepag.es, indexer.coracle.social, user.kindpag.es
- IndexerRelayListEvent を使用してユーザーのリレーリストを取得
- リレーリストがない場合は DefaultIndexerRelayList にフォールバック
- NIP-65 (kind:10002) 対応

## 参考
- https://github.com/vitorpamplona/amethyst/blob/main/amethyst/src/main/java/com/vitorpamplona/amethyst/model/AccountSettings.kt
- https://github.com/vitorpamplona/amethyst/blob/main/amethyst/src/main/java/com/vitorpamplona/amethyst/model/nip51Lists/indexerRelays/IndexerRelayListState.kt
