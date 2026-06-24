← [README](../README.md)

# Damus リレー取得方法

## 結論
- **リレー取得方法**: NIP-65 (kind:10002) を優先。フォールバック順: メモリ内キャッシュ → NIP-65 → kind:3 → UserDefaults → Bootstrapリレー。kind:10002 をリアルタイム購読して更新

## ソースコード

**ファイル**: `damus/Core/Networking/NostrNetworkManager/UserRelayListManager.swift` (行 59-77)

```swift
func getBestEffortRelayList() -> NIP65.RelayList {
    guard let userCurrentRelayList = self.getUserCurrentRelayList() else {
        return NIP65.RelayList(relays: delegate.bootstrapRelays)
    }
    return userCurrentRelayList
}

func getUserCurrentRelayList() -> NIP65.RelayList? {
    if let lastSetRelayList { return lastSetRelayList }
    if let latestRelayListEvent = try? self.getLatestNIP65RelayList() { return latestRelayListEvent }
    if let latestRelayListEvent = try? self.getLatestKind3RelayList() { return latestRelayListEvent }
    if let latestRelayListEvent = try? self.getLatestUserDefaultsRelayList() { return latestRelayListEvent }
    return nil
}
```

## 説明
- `getBestEffortRelayList()` がユーザーの現在のリレーリストを取得し、取得できない場合のみ `delegate.bootstrapRelays`（Bootstrapリレー）にフォールバックする。
- `getUserCurrentRelayList()` のフォールバック順序: メモリ内キャッシュ `lastSetRelayList` → NIP-65 (kind:10002) → kind:3 → UserDefaults。
- 前回調査から refactor され、先頭にメモリ内キャッシュ `lastSetRelayList` が追加された。nostrdb への書き込み遅延を埋め、設定直後でも最新のリストを反映する目的。
- `listenAndHandleRelayUpdates()` で kind:10002 をリアルタイム購読し、より新しいリストのみを反映して更新する。

## 参考
- https://github.com/damus-io/damus/blob/master/damus/Core/Networking/NostrNetworkManager/UserRelayListManager.swift

---
← [README](../README.md)
