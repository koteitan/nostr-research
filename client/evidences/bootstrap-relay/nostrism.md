← [README](../README.md)

# Nostrism Bootstrap Relay

## 結論
- **Bootstrap リレー**: 初回起動時のみ、端末の言語で切り替えてリレー表へシードする。日本語(`ja`)は `relay-jp.shino3.net`（本アプリ運営）, `yabu.me`, `relay.damus.io`, `nos.lol` の4件。それ以外は `relay.damus.io`, `nos.lol` の2件。
- リレー表が空のとき（初回）だけ使われ、既存ユーザーのリレー表は上書きしない。
- 別途 `INDEXER_RELAYS`（purplepag.es, relay.nostr.band, relay.damus.io, nos.lol, relay.primal.net）を kind:0/kind:10002 の確実な取得用に一時接続する。

## ソースコード

**ファイル**: `composeApp/src/commonMain/kotlin/app/nostrdeck/data/DefaultRelays.kt` (行 11-22)

```kotlin
fun defaultRelaysFor(language: String): List<String> = when (language.lowercase().take(2)) {
    "ja" -> listOf(
        "wss://relay-jp.shino3.net",   // 日本向け（本アプリ運営）
        "wss://yabu.me",               // 日本の大手フリーリレー
        "wss://relay.damus.io",        // グローバル
        "wss://nos.lol",               // グローバル
    )
    else -> listOf(
        "wss://relay.damus.io",
        "wss://nos.lol",
    )
}
```

**ファイル**: `composeApp/src/commonMain/kotlin/app/nostrdeck/data/EventRepository.kt` (行 3741-3747)

```kotlin
val INDEXER_RELAYS = listOf(
    "wss://purplepag.es",
    "wss://relay.nostr.band",
    "wss://relay.damus.io",
    "wss://nos.lol",
    "wss://relay.primal.net",
)
```

## 説明
- `defaultRelaysFor()` は「リレー表が空のとき（＝初回起動）」に `EventRepository.start()` からのみ呼ばれ、既定リレーを DB へシードする。既存ユーザーのリレー設定は書き換えない。
- 日本語話者には日本の投稿が多いリレー（自前運営の `relay-jp.shino3.net` と `yabu.me`）を含め、初見のタイムラインが英語圏だけにならないようにしている。非日本語ロケールでは日本リレーを含めない。
- シード後の実際のリレーは NIP-65（kind:10002）でユーザーが編集・公開する（[relay 参照](../relay/nostrism.md)）。フォローがまだ無い新規アカウント向けには、フォロー中ユーザーの NIP-65 を集計する `fetchRelayRecommendations` によるレコメンドが主役で、`DefaultRelays.kt` はその最小フォールバック。
- `INDEXER_RELAYS` はプロフィール(kind:0)やリレーリスト(kind:10002)を確実に引くための一時接続先で、Bootstrap とは別系統。NIP-46 のデフォルトリレーは `nos.lol`（`signer/Nip46.kt`）。

## 参考
- https://github.com/ShinoharaTa/nostr-andloid-native-client/blob/main/composeApp/src/commonMain/kotlin/app/nostrdeck/data/DefaultRelays.kt
- https://github.com/ShinoharaTa/nostr-andloid-native-client/blob/main/composeApp/src/commonMain/kotlin/app/nostrdeck/data/EventRepository.kt

---
← [README](../README.md)
