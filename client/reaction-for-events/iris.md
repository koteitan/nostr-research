← [README](../README.md)

# iris リアクション取得方法

## 結論
- **リアクション取得方法**: イベントごとに #e タグで購読 (closeOnEose)

## ソースコード

**ファイル**: `src/shared/hooks/useReactions.ts` (行 19-58)

```typescript
export function useReactionsByAuthor(eventId: string) {
  const [reactionsByAuthor, setReactionsByAuthor] = useState<Map<string, NDKEvent>>(
    new Map()
  )

  useEffect(() => {
    const filter = {
      kinds: [7],
      ["#e"]: [eventId],
    }

    // Closed on eose because NDK will otherwise send too many concurrent REQs
    const sub = ndk().subscribe(filter, {closeOnEose: true})

    sub?.on("event", (reactionEvent: NDKEvent) => {
      if (shouldHideUser(reactionEvent.author.pubkey)) return

      const authorPubkey = reactionEvent.pubkey

      // Update author's latest reaction
      setReactionsByAuthor((prev) => {
        const existing = prev.get(authorPubkey)
        if (existing && existing.created_at! >= reactionEvent.created_at!) {
          return prev
        }
        const newMap = new Map(prev)
        newMap.set(authorPubkey, reactionEvent)
        return newMap
      })
    })

    return () => {
      sub.stop()
    }
  }, [eventId])

  return reactionsByAuthor
}
```

## 説明

- 各イベントごとに `#e` タグフィルタでリアクションを購読
- `closeOnEose: true` で初期データ取得後に購読を閉じる
- 同じユーザーからの複数リアクションは最新のみ保持
- ミュートユーザーのリアクションはフィルタリング

## 参考
- https://github.com/irislib/iris-client/blob/main/src/shared/hooks/useReactions.ts

---
← [README](../README.md)
