# Amethyst Bootstrap Relay

## 結論
- **Bootstrap Inbox**: relay.nostr.band, relay.damus.io, relay.primal.net, nostr.mom, nos.lol, nostr.bitcoiner.social, nostr.oxtr.dev

## ソースコード

**ファイル**: `amethyst/src/main/java/com/vitorpamplona/amethyst/model/Constants.kt`

```kotlin
object Constants {
    val nos = RelayUrlNormalizer.normalize("wss://nos.lol")
    val mom = RelayUrlNormalizer.normalize("wss://nostr.mom")
    val primal = RelayUrlNormalizer.normalize("wss://relay.primal.net")
    val damus = RelayUrlNormalizer.normalize("wss://relay.damus.io")
    val wine = RelayUrlNormalizer.normalize("wss://nostr.wine")
    val band = RelayUrlNormalizer.normalize("wss://relay.nostr.band")
    val bitcoiner = RelayUrlNormalizer.normalize("wss://nostr.bitcoiner.social")
    val oxtr = RelayUrlNormalizer.normalize("wss://nostr.oxtr.dev")

    val bootstrapInbox = setOf(band, damus, primal, mom, nos, bitcoiner, oxtr)
    val eventFinderRelays = setOf(band, wine, damus, primal, mom, nos, bitcoiner, oxtr)
}
```

## 説明

- bootstrapInbox: 初期接続時のリレー (7リレー)
- eventFinderRelays: イベント検索時の追加リレー (8リレー、wineを含む)
- Outbox モデル対応で、リレー選択を自動化

## 参考
- https://github.com/vitorpamplona/amethyst/blob/main/amethyst/src/main/java/com/vitorpamplona/amethyst/model/Constants.kt
