← [README](../README.md)

# algia 検索リレー

## 結論
- **検索リレー**: 設定ファイルで Search:true(かつ Read:true)を持つリレーを検索に使用。デフォルトでは relay.nostr.band が該当

## ソースコード

**ファイル**: `main.go` (行 716-723)

```go
} else if filter.Search != "" {
	for k, v := range cfg.Relays {
		if !v.Read || !v.Search {
			continue
		}
		rmap[k] = struct{}{}
	}
}
```

## 説明
- Relay 構造体(main.go:42-49)の Search フラグで検索対象リレーを判定する。
- `filter.Search` が指定された場合、config.json の各リレーのうち Read かつ Search が true のものだけを選択する。
- 同等の選択ロジックは main.go:832 にも存在する。
- NIP-50 検索に対応。固定の検索専用リレーはなく、config.json の search:true 設定に依存する。

## 参考
- https://github.com/mattn/algia/blob/main/main.go

---
← [README](../README.md)
