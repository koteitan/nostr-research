← [README](../README.md)

# algia リレー取得方法

## 結論
- **リレー取得方法**: 設定ファイルの relays で初期接続し、その read リレーから kind:10002 (NIP-65 RelayListMetadata) を取得して上書きする

## ソースコード

**ファイル**: `main.go` (行 245-289)

```go
// Get relay list metadata
if !cfg.tempRelay {
	ctx, cancel := context.WithTimeout(context.Background(), relayMetadataTimeout)
	if ev := cfg.pool.QuerySingle(ctx, relays, nostr.Filter{
		Kinds:   []int{nostr.KindRelayListMetadata},
		Authors: []string{pub},
		Limit:   1,
	}); ev != nil {
		rm := map[string]Relay{}
		for _, r := range ev.Tags.GetAll([]string{"r"}) {
			...
		}
		...
		if len(rm) > 0 {
			cfg.Relays = rm
		}
	}
}
```

## 説明
- 設定ファイルの `relays` で初期接続し、その read リレーへ問い合わせる。
- kind:10002 (NIP-65 RelayListMetadata) の `r` タグから read/write を判定する。
- kind:10002 が存在し、構築した `rm` が非空のときのみ `cfg.Relays` を上書きする。
- 既存リレーの Search フラグは引き継ぐ。
- `--relay` 指定時 (tempRelay) はメタデータを取得しない。

## 参考
- https://github.com/mattn/algia/blob/main/main.go

---
← [README](../README.md)
