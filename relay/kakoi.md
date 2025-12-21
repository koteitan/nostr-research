# kakoi リレー取得方法

## 結論
- **リレー取得方法**: 設定から取得 (relays.json)

## ソースコード

**ファイル**: `kakoi/Tools.cs` (行 182-213)

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
    try
    {
        var jsonContent = File.ReadAllText(_relaysJsonPath);
        var relays = JsonSerializer.Deserialize<List<Relay>>(jsonContent, GetOption());
        if (relays != null)
        {
            return relays;
        }
        return [];
    }
    // ...
}
```

**リレー構造** (Tools.cs, 行 37-43):

```csharp
public class Relay
{
    [JsonPropertyName("enabled")]
    public bool Enabled { get; set; }
    [JsonPropertyName("url")]
    public string? Url { get; set; }
}
```

## 説明

- kind:10002 からは取得していない
- 設定ファイル (`relays.json`) から読み込む方式
- ファイルが存在しない場合はデフォルトリレーを使用

## 参考
- https://github.com/betonetojp/kakoi
