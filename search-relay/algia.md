# algia 検索リレー

## 結論
- **検索リレー**: relay.nostr.band（search フラグ）

## ソースコード

**ファイル**: `main.go` (行 666-673)

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

**リレー構造体定義** (行 32-40):

```go
type Relay struct {
    Read     bool `json:"read"`
    Write    bool `json:"write"`
    Search   bool `json:"search"`    // 検索用フラグ
    Global   bool `json:"global"`
    DM       bool `json:"dm"`
    Bookmark bool `json:"bm"`
}
```

## 説明

- 設定ファイルにおいて、各リレーに対して `"search": true` と明示的に設定されたリレーが検索対象
- `relay.nostr.band` はデフォルトで `Search: true` で設定される

## 参考
- https://github.com/mattn/algia/blob/main/main.go
