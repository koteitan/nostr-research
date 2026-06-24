← [README](../README.md)

# Coracle リアクション取得方法

## 結論
- **リアクション取得方法**: ノート表示時に各イベント単位で取得（per-note）。`Note.svelte` の `onMount` で `note_actions` 設定に応じ REACTION(7)/ZAP_RESPONSE(9735) 等を `Router.Replies(event)` のリレーへ購読し、NoteReducer が context へ集約

## ソースコード

**ファイル**: `src/app/shared/Note.svelte` (行 80-103)

```svelte
onMount(() => {
  ...
  const actions = getSetting("note_actions")
  const kinds = []
  if (actions.includes("replies")) { kinds.push(NOTE); kinds.push(COMMENT) }
  if (actions.includes("reactions")) { kinds.push(REACTION) }
  if (env.ENABLE_ZAPS && actions.includes("zaps")) { kinds.push(ZAP_RESPONSE) }

  myLoad({
    relays: Router.get().Replies(event).policy(addMaximalFallbacks).getUrls(),
    filters: getReplyFilters([event], {kinds}),
  })
})
```

## 説明
- ノートコンポーネント `Note.svelte` の `onMount` 時に、各イベント単位（per-note）でリアクション等を購読する。
- `note_actions` 設定（`getSetting("note_actions")`）に応じて購読する kind を組み立てる。`reactions` 有効時に REACTION(7)、`zaps` 有効かつ `env.ENABLE_ZAPS` で ZAP_RESPONSE(9735) を追加。
- 購読先リレーは `Router.get().Replies(event)` に `addMaximalFallbacks` ポリシーを適用したものを使い、フィルターは `getReplyFilters([event], {kinds})` で生成する。
- `reactionKinds = [REACTION, ZAP_RESPONSE]` は `src/util/nostr.ts`（73 行）で定義されている。
- `NoteReducer.svelte`（75-108 行）がリポスト/リアクション/Zap を親イベントに紐付けて context(Map) に格納し、`NoteReactions.svelte` が pubkey で重複排除して表示する。

## 参考
- https://github.com/coracle-social/coracle/blob/master/src/app/shared/Note.svelte

---
← [README](../README.md)
