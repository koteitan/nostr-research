← [README](../README.md)

# Amethyst リレー取得方法

## 結論
- **リレー取得方法**: Outbox モデル (NIP-65 kind:10002)。ホームタイムライン用の書き込み/読み込みリレーは `AdvertisedRelayListEvent` から取得し `outboxFlow` / `inboxFlow` で公開。
- 未設定時は `Constants.eventFinderRelays` / `bootstrapInbox` にフォールバック。
- 加えて Indexer リレー (`DefaultIndexerRelayList`: purplepag.es, indexer.coracle.social, user.kindpag.es, directory.yabu.me, profiles.nostr1.com) で他ユーザーのリレーリスト等を取得。

## ソースコード

**ファイル**: `amethyst/src/main/java/com/vitorpamplona/amethyst/model/nip65RelayList/Nip65RelayListState.kt` (行 61-93)

```kotlin
fun normalizeNIP65WriteRelayListWithBackup(note: Note): Set<NormalizedRelayUrl> = nip65Event(note)?.writeRelaysNorm()?.toSet() ?: Constants.eventFinderRelays

fun normalizeNIP65ReadRelayListWithBackup(note: Note): Set<NormalizedRelayUrl> = nip65Event(note)?.readRelaysNorm()?.toSet() ?: Constants.bootstrapInbox

val outboxFlow =
    getNIP65RelayListFlow()
        .map { normalizeNIP65WriteRelayListWithBackup(it.note) }
        .onStart { emit(normalizeNIP65WriteRelayListWithBackup(nip65ListNote)) }
```

## 説明
- NIP-65 (kind:10002) のアウトボックスモデルを採用。
- 書き込みリレーは `writeRelaysNorm()`、読み込みリレーは `readRelaysNorm()` から取得し、`outboxFlow` / `inboxFlow` として公開する。
- ユーザーが NIP-65 リレーリストを未設定の場合、書き込みは `Constants.eventFinderRelays`、読み込みは `Constants.bootstrapInbox` にフォールバックする。
- `DefaultIndexerRelayList` は `commons/defaults/AmethystDefaults.kt` (62-63行) に移動し、旧リストに directory.yabu.me と profiles.nostr1.com が追加された。
- 他ユーザーのリレーリスト等の取得は Indexer リレー経由で行い、`IndexerRelayListState.normalizeIndexerRelayListWithBackup()` で処理する。

## 参考
- https://github.com/vitorpamplona/amethyst/blob/main/amethyst/src/main/java/com/vitorpamplona/amethyst/model/nip65RelayList/Nip65RelayListState.kt

---
← [README](../README.md)
