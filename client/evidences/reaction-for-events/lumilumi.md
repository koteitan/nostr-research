← [README](../README.md)

# Lumilumi リアクション取得方法

## 結論
- **リアクション取得方法**: リアクション取得は「表示中イベントをまとめてバッチ購読」。viewEventIds に集めた表示中ノートの e/a タグを 1 秒デバウンスで集約し、kinds [7,6,16] (リアクション/リポスト) と [9735] (zap receipt) を #e/#a でまとめてフィルタ化、専用 rx-nostr インスタンス rxNostr3 (forward req) に emit して継続購読する。

## ソースコード

**ファイル**: `src/lib/components/renderSnippets/nostr/SetRepoReactions.svelte` (行 71-121)

```ts
function performUpdate() {
  const now = Math.floor(Date.now() / 1000);
  filters = etagList.length > 0 ? [
    { "#e": $state.snapshot(etagList),
      authors: lumiSetting.get().showAllReactions ? undefined : [lumiSetting.get().pubkey],
      kinds: [7, 6, 16], limit: 0, since: now },
    { "#e": $state.snapshot(etagList), kinds: [9735], limit: 0 },
  ] : [];
  if (atagList.length > 0) {
    filters.push(
      { "#a": $state.snapshot(atagList), ..., kinds: [7, 6, 16], limit: 0, since: now },
      { "#a": $state.snapshot(atagList), kinds: [9735], limit: 0, since: now });
  }
  changeEmit(filters);
}
```

## 説明
- `viewEventIds` に集めた表示中ノートの e/a タグを 1 秒デバウンスで集約し、`performUpdate()` で `#e`/`#a` フィルタを生成する。
- kinds [7, 6, 16] (リアクション/リポスト) と [9735] (zap receipt) を `#e`/`#a` でまとめてフィルタ化する。
- `changeEmit` → 専用の reaction/repost 用 rx-nostr インスタンス `rxNostr3` (forward req) へ emit して継続購読する。
- 受信は `src/lib/func/reactions.ts` の `useReq3`/`handleEvent` で kind 7=reaction, 6/16=repost, 9735=zapped を `queryClient` にキャッシュする。
- 投稿ごとではなく表示中ノートをまとめて 1 秒デバウンスでバッチ購読する。スレッド全リアクション集計は `useAllReactions.ts` も併用する。

## 参考
- https://github.com/TsukemonoGit/lumilumi/blob/main/src/lib/components/renderSnippets/nostr/SetRepoReactions.svelte

---
← [README](../README.md)
