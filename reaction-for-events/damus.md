# Damus リアクション取得方法

## 結論
- **リアクション取得方法**: #p フィルタで自分宛の通知を購読 (リアルタイム)

## ソースコード

**ファイル**: `damus/Features/Timeline/Models/HomeModel.swift` (行 506-534)

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

// Subscribe to notifications
self.notificationsHandlerTask = Task {
    for await item in damus_state.nostrNetwork.reader.advancedStream(
        filters: notifications_filters,
        to: nil
    ) {
        // ...
    }
}
```

**ファイル**: `damus/Features/Notifications/Models/NotificationsModel.swift` (行 211-222)

```swift
private func insert_reaction(_ ev: NostrEvent) -> Bool {
    guard let id = ev.referenced_ids.last else {
        return false
    }

    if let evgrp = self.reactions[id] {
        return evgrp.insert(ev)
    } else {
        let evgrp = EventGroup()
        self.reactions[id] = evgrp
        return evgrp.insert(ev)
    }
}
```

## 説明

- `#p` (pubkeys) フィルタで自分宛の通知を購読
- kind:7 (like/reaction), kind:6 (boost), kind:9735 (zap), kind:1 (reply) を同時取得
- `onlyzaps_mode` 設定でリアクション非表示可能
- リアクションは `referenced_ids.last` から元イベントを特定
- EventGroup で同一イベントへのリアクションをグルーピング
- limit: 500 で初期ロード

## 参考
- https://github.com/damus-io/damus/blob/master/damus/Features/Timeline/Models/HomeModel.swift
- https://github.com/damus-io/damus/blob/master/damus/Features/Notifications/Models/NotificationsModel.swift
