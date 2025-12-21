# kakoi Bootstrap Relay

## 結論
- **Bootstrap リレー**: yabu.me, r.kojira.io, relay-jp.nostr.wirednet.jp, nos.lol, relay.damus.io, relay.nostr.band

## ソースコード

**ファイル**: `kakoi/Tools.cs` (行 182-191)

```csharp
internal static List<Relay> LoadRelays()
{
    List<Relay> defaultRelays = [
        new Relay { Enabled = true, Url = "wss://yabu.me/" },
        new Relay { Enabled = true, Url = "wss://r.kojira.io/" },
        new Relay { Enabled = true, Url = "wss://relay-jp.nostr.wirednet.jp/" },
        new Relay { Enabled = true, Url = "wss://nos.lol/" },
        new Relay { Enabled = true, Url = "wss://relay.damus.io/" },
        new Relay { Enabled = true, Url = "wss://relay.nostr.band/" },
    ];

    // relays.json を読み込み
    if (!File.Exists(_relaysJsonPath))
    {
        return defaultRelays;  // ファイルがなければデフォルトリレーを返す
    }
    // ...
}
```

## 説明

- 設定ファイル (`relays.json`) が存在しない場合、上記のデフォルトリレーを使用
- 設定ファイルが存在する場合はそちらを優先

## 参考
- https://github.com/betonetojp/kakoi
