← [README](../README.md)

# kakoi Bootstrap Relay

## 結論
- **Bootstrap リレー**: ハードコードされたデフォルトリレー 6 件 (yabu.me, r.kojira.io, relay-jp.nostr.wirednet.jp, nos.lol, relay.damus.io, relay.nostr.band)。relays.json が無い場合に使用。

## ソースコード

**ファイル**: `kakoi/Tools.cs` (行 184-191)

```csharp
List<Relay> defaultRelays = [
    new Relay { Enabled = true, Url = "wss://yabu.me/" },
    new Relay { Enabled = true, Url = "wss://r.kojira.io/" },
    new Relay { Enabled = true, Url = "wss://relay-jp.nostr.wirednet.jp/" },
    new Relay { Enabled = true, Url = "wss://nos.lol/" },
    new Relay { Enabled = true, Url = "wss://relay.damus.io/" },
    new Relay { Enabled = true, Url = "wss://relay.nostr.band/" },
    ];
```

## 説明
- `LoadRelays()` 内で、`relays.json` が存在しない場合に返す日本語圏中心の固定リスト。
- yabu.me / r.kojira.io / relay-jp.nostr.wirednet.jp など日本リレーを含む 6 件で構成。
- ロケール分岐や環境変数による追加は無し。

## 参考
- https://github.com/betonetojp/kakoi/blob/master/kakoi/Tools.cs

---
← [README](../README.md)
