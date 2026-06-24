← [README](../README.md)

# Amethyst 検索リレー

## 結論
- **検索リレー**: 全文検索 (NIP-50)。SearchRelayListEvent (kind:10007) からユーザー設定の検索リレーを取得し、未設定時は DefaultSearchRelayList (nostr.wine, relay.noswhere.com, search.nos.today, antiprimal.net, relay.ditto.pub) にフォールバック。

## ソースコード

**ファイル**: `commons/src/commonMain/kotlin/com/vitorpamplona/amethyst/commons/defaults/AmethystDefaults.kt` (行 59-60)

```kotlin
val DefaultSearchRelayList =
    setOf(Constants.wine, Constants.where, Constants.nostoday, Constants.antiprimal, Constants.ditto)
```

## 説明

- 検索リレーは NIP-50 (全文検索) を利用する
- ユーザー設定は SearchRelayListEvent (kind:10007) から取得する
- 未設定時は `DefaultSearchRelayList` の5リレーにフォールバックする
- フォールバック適用は `SearchRelayListState.normalizeSearchRelayListWithBackup()` (61行)
- 5リレーに変更され、旧リストにあった relay.nostr.band が外れ antiprimal.net と relay.ditto.pub が追加された
- `where`=relay.noswhere.com、`nostoday`=search.nos.today

## 参考
- https://github.com/vitorpamplona/amethyst/blob/main/commons/src/commonMain/kotlin/com/vitorpamplona/amethyst/commons/defaults/AmethystDefaults.kt

---
← [README](../README.md)
