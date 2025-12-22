← [README](../README.md)

# Damus リレー取得方法

## 結論
- **リレー取得方法**: NIP-65 (kind:10002) + kind:3 フォールバック

## ソースコード

**ファイル**: `damus/Core/Networking/NostrNetworkManager/UserRelayListManager.swift`

```swift
/// Gets the user's current relay list.
///
/// It attempts to get a NIP-65 relay list from the local database, or falls back to a legacy list.
func getUserCurrentRelayList() -> NIP65.RelayList? {
    if let latestRelayListEvent = try? self.getLatestNIP65RelayList() { return latestRelayListEvent }
    if let latestRelayListEvent = try? self.getLatestKind3RelayList() { return latestRelayListEvent }
    if let latestRelayListEvent = try? self.getLatestUserDefaultsRelayList() { return latestRelayListEvent }
    return nil
}

/// Gets the "best effort" relay list.
func getBestEffortRelayList() -> NIP65.RelayList {
    guard let userCurrentRelayList = self.getUserCurrentRelayList() else {
        return NIP65.RelayList(relays: delegate.bootstrapRelays)
    }
    return userCurrentRelayList
}

func listenAndHandleRelayUpdates() async {
    let filter = NostrFilter(kinds: [.relay_list], authors: [delegate.keypair.pubkey])
    for await noteLender in self.reader.streamIndefinitely(filters: [filter]) {
        // ...
        guard let relayList = try? NIP65.RelayList(event: note) else { continue }
        try? await self.set(userRelayList: relayList)
    }
}
```

## 説明

- NIP-65 (kind:10002) を優先
- フォールバック順: NIP-65 → kind:3 → UserDefaults → Bootstrap
- リレーリスト変更をリアルタイム購読
- read/write 設定を分離管理

## 参考
- https://github.com/damus-io/damus/blob/master/damus/Core/Networking/NostrNetworkManager/UserRelayListManager.swift

---
← [README](../README.md)
