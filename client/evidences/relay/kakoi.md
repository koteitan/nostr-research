← [README](../README.md)

# kakoi リレー取得方法

## 結論
- **リレー取得方法**: 設定ファイル relays.json から取得（無ければデフォルトリレー）。kind:10002 (NIP-65) は使用しない。

## ソースコード

**ファイル**: `kakoi/Tools.cs` (行 182-213)

```csharp
internal static List<Relay> LoadRelays()
{
    List<Relay> defaultRelays = [ ... ];
    // relays.jsonを読み込み
    if (!File.Exists(_relaysJsonPath))
    {
        return defaultRelays;
    }
    var jsonContent = File.ReadAllText(_relaysJsonPath);
    var relays = JsonSerializer.Deserialize<List<Relay>>(jsonContent, GetOption());
    if (relays != null)
    {
        return relays;
    }
    return [];
}
```

## 説明
- ホームタイムラインのリレーは StartupPath の `relays.json` から取得する。
- `relays.json` が存在しない場合はコード内のデフォルトリレー一覧を返す。
- `GetEnabledRelays`（行 215-227）が `enabled=true` のリレーのみを抽出する。
- kind:10002 や outbox モデルによる探索は無く、ユーザーが手動編集する固定リスト方式。

## 参考
- https://github.com/betonetojp/kakoi/blob/master/kakoi/Tools.cs

---
← [README](../README.md)
