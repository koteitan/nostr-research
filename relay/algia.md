# algia リレー取得方法

## 結論
- **リレー取得方法**: kind:10002 (NIP-65)
- 設定ファイルで初期接続、kind:10002 で上書き

## ソースコード

**ファイル**: `main.go` (行 168-230)

```go
// Get relay list metadata
if !cfg.tempRelay {
    if ev := cfg.pool.QuerySingle(ctx, relays, nostr.Filter{
        Kinds:   []int{nostr.KindRelayListMetadata},  // kind 10002
        Authors: []string{pub},
        Limit:   1,
    }); ev != nil {
        rm := map[string]Relay{}
        for _, r := range ev.Tags.GetAll([]string{"r"}) {
            if len(r) == 2 {
                rm[r[1]] = Relay{
                    Read:  true,
                    Write: true,
                }
            } else if len(r) == 3 {
                switch r[2] {
                case "read":
                    rm[r[1]] = Relay{
                        Read:  true,
                        Write: false,
                    }
                case "write":
                    rm[r[1]] = Relay{
                        Read:  true,
                        Write: true,
                    }
                }
            }
        }
        cfg.Relays = rm
    }
}
```

## 処理フロー

1. まず設定ファイル (`~/.config/algia/config.json`) からリレーを読み込む
2. kind:10002 (RelayListMetadata) イベントをクエリ
3. kind:10002 が存在すればそれで既存の設定を上書き
4. kind:10002 内の「r」タグからリレー URL 情報を抽出
5. 「read」「write」フラグで読み書き権限を判定

## 参考
- https://github.com/mattn/algia/blob/main/main.go
