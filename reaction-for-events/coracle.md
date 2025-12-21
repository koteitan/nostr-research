# Coracle リアクション取得方法

## 結論
- **リアクション取得方法**: フィード購読時にコンテキストとして取得 (NoteReducer)

## ソースコード

**ファイル**: `src/util/nostr.ts` (行 71)

```typescript
export const reactionKinds = [REACTION, ZAP_RESPONSE] as number[]
```

**ファイル**: `src/app/shared/NoteReducer.svelte` (行 67-97)

```svelte
const addEvent = async (event: TrustedEvent) => {
  // ...
  while (currentDepth > 0) {
    const parent = await getParent(event)

    // Skip zaps that fail our zapper check
    if (event.kind === ZAP_RESPONSE && !(await getValidZap(event, parent))) {
      return
    }

    // Link the events, even if we end up skipping this one
    addToMapKey(context, getIdOrAddress(parent), event)
    // ...
  }
}
```

**ファイル**: `src/app/shared/NoteReactions.svelte` (行 18-21)

```svelte
$: reposts = context.filter(e => repostKinds.includes(e.kind))
$: reactions = uniqBy(prop("pubkey"), context.filter(spec({kind: REACTION})))
$: zaps = deriveValidZaps(context.filter(spec({kind: ZAP_RESPONSE})), event)
```

## 説明

- フィード読み込み時に @welshman/app の repository を使用
- NoteReducer がイベント間の親子関係を構築
- context (Map) にリアクション・リポスト・Zap を格納
- リアクションは pubkey で重複排除
- 個別のイベントごとに #e 取得ではなく、ストリームで受信

## 参考
- https://github.com/coracle-social/coracle/blob/master/src/app/shared/NoteReducer.svelte
- https://github.com/coracle-social/coracle/blob/master/src/app/shared/NoteReactions.svelte
