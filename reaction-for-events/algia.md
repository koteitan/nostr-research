← [README](../README.md)

# algia リアクション取得方法

## 結論
- **リアクション取得方法**: リアクション取得なし（送信のみ）

## ソースコード

**ファイル**: `timeline.go` (行 494-583)

```go
func doLike(cCtx *cli.Context) error {
    return callLike(&likeArg{
        cfg:     cCtx.App.Metadata["config"].(*Config),
        id:      cCtx.String("id"),
        content: cCtx.String("content"),
        emoji:   cCtx.String("emoji"),
    })
}

func callLike(arg *likeArg) error {
    // ...
    ev.Kind = nostr.KindReaction  // kind 7
    ev.Content = arg.content
    if arg.emoji != "" {
        if ev.Content == "" {
            ev.Content = "like"
        }
        ev.Tags = ev.Tags.AppendUnique(nostr.Tag{"emoji", ev.Content, arg.emoji})
        ev.Content = ":" + ev.Content + ":"
    }
    if ev.Content == "" {
        ev.Content = "+"
    }
    // ...
}
```

## 説明

- リアクション送信機能: `algia like --id [note-id]`
- リアクション削除機能: `algia unlike --id [note-id]`
- カスタム絵文字リアクション対応（`--emoji` フラグ）
- **他のユーザーのリアクション取得・表示機能は実装されていない**

## 参考
- https://github.com/mattn/algia/blob/main/timeline.go

---
← [README](../README.md)
