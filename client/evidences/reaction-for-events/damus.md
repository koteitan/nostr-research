← [README](../README.md)

# Damus リアクション取得方法

## 結論
- **リアクション取得方法**: #p (pubkeys) フィルタで自分宛の通知をリアルタイム購読し収集。kind:7(like)/6(boost)/9735(zap)/1(reply) を同時取得

## ソースコード

**ファイル**: `damus/Features/Timeline/Models/HomeModel.swift` (行 563-609)

```swift
var notifications_filter_kinds: [NostrKind] = [
    .text,
    .boost,
    .zap,
]
if !damus_state.settings.onlyzaps_mode {
    notifications_filter_kinds.append(.like)
}
var notifications_filter = NostrFilter(kinds: notifications_filter_kinds)
notifications_filter.pubkeys = [damus_state.pubkey]
notifications_filter.limit = 500

let notifications_filters = [notifications_filter]
...
self.notificationsHandlerTask = Task {
    for await item in damus_state.nostrNetwork.reader.advancedStream(
        filters: notifications_filters,
        streamMode: .ndbAndNetworkParallel(networkOptimization: .sinceOptimization)
    ) {
        switch item {
        case .event(let lender):
            await lender.justUseACopy({ await process_event(ev: $0, context: .notifications) })
        case .ndbEose:
            ...
        }
    }
}
```

## 説明
- 自分の pubkey を `notifications_filter.pubkeys` に設定し、#p タグで自分宛の通知イベントのみを購読する。
- 取得対象 kind は text(1, reply)/boost(6)/zap(9735) と like(7)。like は `onlyzaps_mode` 設定が有効な場合は除外される。
- `advancedStream` + `.ndbAndNetworkParallel(networkOptimization: .sinceOptimization)` でローカル DB とネットワークを並行購読し、リアルタイムに通知を収集する。
- 前回調査時の `streamIndefinitely` から `advancedStream` へ変更。EOSE を受け取り、ローカル DB ロード完了後に通知をフラッシュする競合修正が行われている。
- リアクションは `NotificationsModel.insert_reaction` (211-223) で `referenced_ids.last` から元イベントを特定し、`EventGroup` でグルーピングされる。
- `limit: 500` で初期ロードする。

## 参考
- https://github.com/damus-io/damus/blob/master/damus/Features/Timeline/Models/HomeModel.swift

---
← [README](../README.md)
