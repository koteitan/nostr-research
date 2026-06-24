← [README](../README.md)

# iris リアクション取得方法

## 結論
- **リアクション取得方法**: イベントごとに #e タグで kind:7 を購読し closeOnEose: true で取得 (useReactionsByAuthor)。同一作者は最新のみ保持。

## ソースコード

**ファイル**: `src/shared/hooks/useReactions.ts` (行 19-58)

```typescript
export function useReactionsByAuthor(eventId: string) {
  ...
  useEffect(() => {
    const filter = {
      kinds: [7],
      ["#e"]: [eventId],
    }
    // Closed on eose because NDK will otherwise send too many concurrent REQs ...
    const sub = ndk().subscribe(filter, {closeOnEose: true})
    sub?.on("event", (reactionEvent: NDKEvent) => {
      if (shouldHideUser(reactionEvent.author.pubkey)) return
      ...
    })
    return () => { sub.stop() }
  }, [eventId])
```

## 説明
- 個別投稿のリアクション収集は `useReactionsByAuthor` が担当し、`#e` タグに対象イベント ID を指定して kind:7 を購読する。
- `closeOnEose: true` を指定することで EOSE 受信時に購読を閉じ、NDK が大量の同時 REQ を送らないようにしている。
- 同一作者からのリアクションは最新のものだけを保持する。
- ミュート対象ユーザー (`shouldHideUser`) のリアクションは除外される。
- 別途 `src/shared/hooks/useReactionSubscription.ts` はフィードの人気判定用に kind:7/6 をまとめて購読する仕組みで、用途が異なる。

## 参考
- https://github.com/irislib/iris-client/blob/main/src/shared/hooks/useReactions.ts

---
← [README](../README.md)
