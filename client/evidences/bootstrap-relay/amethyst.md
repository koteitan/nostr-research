← [README](../README.md)

# Amethyst Bootstrap Relay

## 結論
- **Bootstrap リレー**: `Constants.bootstrapInbox` の7リレー (nos.lol, nostr.mom, relay.primal.net, relay.damus.io, nostr.bitcoiner.social, nostr.oxtr.dev, directory.yabu.me) を初期接続のフォールバックに使用。
- NIP-65 読み込みリレーが未取得のとき `normalizeNIP65ReadRelayListWithBackup()` が `bootstrapInbox` を返す。

## ソースコード

**ファイル**: `commons/src/commonMain/kotlin/com/vitorpamplona/amethyst/commons/defaults/Constants.kt` (行 30-60)

```kotlin
object Constants {
    val nos = RelayUrlNormalizer.normalize("wss://nos.lol")
    val mom = RelayUrlNormalizer.normalize("wss://nostr.mom")
    val primal = RelayUrlNormalizer.normalize("wss://relay.primal.net")
    val damus = RelayUrlNormalizer.normalize("wss://relay.damus.io")
    val bitcoiner = RelayUrlNormalizer.normalize("wss://nostr.bitcoiner.social")
    val oxtr = RelayUrlNormalizer.normalize("wss://nostr.oxtr.dev")
    val yabu = RelayUrlNormalizer.normalize("wss://directory.yabu.me")

    val bootstrapInbox = setOf(damus, primal, mom, nos, bitcoiner, oxtr, yabu)
    val eventFinderRelays = setOf(wine, damus, primal, mom, nos, bitcoiner, oxtr)
}
```

## 説明
- `Constants.kt` が `amethyst/model` から `commons/defaults` へ移動した。
- 旧 `bootstrapInbox` にあった `relay.nostr.band` が外れ `directory.yabu.me` が追加され、計7リレーとなった。
- `eventFinderRelays` は `nostr.wine` を含む別セットで、用途が異なる。
- フォールバックの消費は `Nip65RelayListState.normalizeNIP65ReadRelayListWithBackup()` (63行) で行われ、NIP-65 読み込みリレーが未取得のとき `bootstrapInbox` を返す。

## 参考
- https://github.com/vitorpamplona/amethyst/blob/main/commons/src/commonMain/kotlin/com/vitorpamplona/amethyst/commons/defaults/Constants.kt

---
← [README](../README.md)
