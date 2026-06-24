← [README](../README.md)

# algia Bootstrap Relay

## 結論
- **Bootstrap リレー**: 設定ファイルにリレー未定義の場合、wss://relay.nostr.band を Read/Write/Search 全フラグ有効でデフォルト追加する

## ソースコード

**ファイル**: `main.go` (行 138-145)

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
- ブートストラップは設定ファイル (`~/.config/algia/config.json`) の `relays` を使用する。
- 設定ファイルの `relays` が空の場合のみ、`wss://relay.nostr.band` を 1 件だけデフォルト追加する。
- 追加されるリレーは Read / Write / Search の全フラグが有効。
- ロケール別の追加リレーは無し。

## 参考
- https://github.com/mattn/algia/blob/main/main.go

---
← [README](../README.md)
