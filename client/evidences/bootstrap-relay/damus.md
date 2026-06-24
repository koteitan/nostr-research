← [README](../README.md)

# Damus Bootstrap Relay

## 結論
- **Bootstrap リレー**: ハードコードされたグローバルBootstrapリレー4つ (relay.damus.io, nostr.land, nostr.wine, nos.lol) に加え、ユーザーのLocale地域に応じて地域別リレーを追加
- 日本語設定時追加: relay-jp.nostr.wirednet.jp, yabu.me, r.kojira.io

## ソースコード

**ファイル**: `damus/Features/Relays/Models/RelayBootstrap.swift` (行 11-74)

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
    ...
]

func get_default_bootstrap_relays() -> [RelayURL] {
    var default_bootstrap_relays: [RelayURL] = BOOTSTRAP_RELAYS.compactMap({ RelayURL($0) })
    if let user_region = Locale.current.region, let regional_bootstrap_relays = REGION_SPECIFIC_BOOTSTRAP_RELAYS[user_region] {
        default_bootstrap_relays.append(contentsOf: regional_bootstrap_relays.compactMap({ RelayURL($0) }))
    }
    return default_bootstrap_relays
}
```

## 説明
- グローバル4リレー (relay.damus.io, nostr.land, nostr.wine, nos.lol) を `BOOTSTRAP_RELAYS` にハードコード
- `REGION_SPECIFIC_BOOTSTRAP_RELAYS` でLocale地域ごとの追加リレーを定義 (日本: relay-jp.nostr.wirednet.jp / yabu.me / r.kojira.io、タイ、ドイツ)
- `get_default_bootstrap_relays()` がグローバル4リレーに `Locale.current.region` に対応する地域別リレーを追加して返す
- `save_bootstrap_relays` / `load_bootstrap_relays` により UserDefaults へカスタムリレーを保存可能

## 参考
- https://github.com/damus-io/damus/blob/master/damus/Features/Relays/Models/RelayBootstrap.swift

---
← [README](../README.md)
