← [README](../README.md)

# Damus Bootstrap Relay

## 結論
- **Bootstrap リレー**: relay.damus.io, nostr.land, nostr.wine, nos.lol
- **地域別リレー**: 日本 (relay-jp.nostr.wirednet.jp, yabu.me, r.kojira.io), タイ, ドイツ

## ソースコード

**ファイル**: `damus/Features/Relays/Models/RelayBootstrap.swift`

```swift
fileprivate let BOOTSTRAP_RELAYS = [
    "wss://relay.damus.io",
    "wss://nostr.land",
    "wss://nostr.wine",
    "wss://nos.lol",
]

fileprivate let REGION_SPECIFIC_BOOTSTRAP_RELAYS: [Locale.Region: [String]] = [
    Locale.Region.japan: [
        "wss://relay-jp.nostr.wirednet.jp",
        "wss://yabu.me",
        "wss://r.kojira.io",
    ],
    Locale.Region.thailand: [
        "wss://relay.siamstr.com",
        "wss://relay.zerosatoshi.xyz",
        "wss://th2.nostr.earnkrub.xyz",
    ],
    Locale.Region.germany: [
        "wss://nostr.einundzwanzig.space",
        "wss://nostr.cercatrova.me",
        "wss://nostr.bitcoinplebs.de",
    ]
]

func get_default_bootstrap_relays() -> [RelayURL] {
    var default_bootstrap_relays: [RelayURL] = BOOTSTRAP_RELAYS.compactMap({ RelayURL($0) })

    if let user_region = Locale.current.region,
       let regional_bootstrap_relays = REGION_SPECIFIC_BOOTSTRAP_RELAYS[user_region] {
        default_bootstrap_relays.append(contentsOf: regional_bootstrap_relays.compactMap({ RelayURL($0) }))
    }

    return default_bootstrap_relays
}
```

## 説明

- グローバルリレー4つ + 地域別リレー
- ユーザーのLocaleに基づいて地域リレーを追加
- 日本、タイ、ドイツの地域リレーをサポート
- UserDefaults にカスタムリレーを保存可能

## 参考
- https://github.com/damus-io/damus/blob/master/damus/Features/Relays/Models/RelayBootstrap.swift

---
← [README](../README.md)
