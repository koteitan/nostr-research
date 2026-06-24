← [README](../README.md)

# algia リアクション取得方法

## 結論
- **リアクション取得方法**: 他者のリアクション取得・表示は無し（送信専用）。like/unlike で kind:7 を発行/削除するのみ

## ソースコード

**ファイル**: `builders.go` (行 33-61)

```go
// buildLikeEvent constructs an unsigned kind 7 (reaction) event.
func buildLikeEvent(pubkey, targetID, relayHint, content, emoji string, mentionedPubkeys []string, now nostr.Timestamp) (*nostr.Event, error) {
	...
	ev := &nostr.Event{
		PubKey:    pubkey,
		CreatedAt: now,
		Kind:      nostr.KindReaction,
		Tags:      nostr.Tags{etag},
		Content:   content,
	}
	...
}
```

## 説明
- like コマンド（timeline.go:349 doLike / 367 callLike）で kind:7 を送信し、--emoji でカスタム絵文字に対応する。
- kind:7 を読む箇所（timeline.go:489 callUnlike）は自分のリアクションを Authors:pub で検索し NIP-09 で削除するためのものである。
- タイムライン上の他者リアクションを集計・表示する実装は存在しない（送信専用）。

## 参考
- https://github.com/mattn/algia/blob/main/builders.go

---
← [README](../README.md)
