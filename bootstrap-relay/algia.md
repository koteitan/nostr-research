← [README](../README.md)

# algia Bootstrap Relay

## 結論
- **Bootstrap リレー**: relay.nostr.band

## ソースコード

**ファイル**: `main.go` (行 128-135)

```go
if len(cfg.Relays) == 0 {
    cfg.Relays = map[string]Relay{}
    cfg.Relays["wss://relay.nostr.band"] = Relay{
        Read:   true,
        Write:  true,
        Search: true,
    }
}
```

## 説明

- 設定ファイル (`~/.config/algia/config.json`) にリレーが定義されていない場合、`relay.nostr.band` がデフォルトとして設定される
- このリレーは Read, Write, Search すべてのフラグが有効

## 参考
- https://github.com/mattn/algia/blob/main/main.go

---
← [README](../README.md)
