← [README](../README.md)

# Lumilumi リレー取得方法

## 結論
- **リレー取得方法**: ホームタイムライン用リレーは「ローカル設定優先 → kind:10002 (NIP-65) → defaultRelays フォールバック」。`lumiSetting.useRelaySet !== "0"` ならユーザのローカル設定(`lumiSetting.relays`)を使用、`"0"` なら relaySearchRelays 経由で kind:10002 をフェッチしてユーザのリレーセットを適用、見つからなければ defaultRelays。
- kind:10002 が無い旧形式は kind:3 content からも解釈。

## ソースコード

**ファイル**: `src/lib/components/renderSnippets/nostr/relay/SetDefaultRelays.svelte` (行 228-285)

```svelte
async function initWithDefaultRelays(): Promise<void> {
  if (!pubkey) { await applyRelays(defaultRelays); ... return; }
  if (lumiSetting.get().useRelaySet !== "0") {
    await applyRelays(lumiSetting.get().relays); ... return;
  }
  // useRelaySet === "0": kind:10002から取得
  eventPacket = await fetchKind10002(...);
  if (eventPacket) {
    const relays = setRelaysByKind10002(eventPacket.event);
    await applyRelays(relays);
  } else {
    // kind:10002 が見つからなかった場合、defaultRelays にフォールバック
    await applyRelays(defaultRelays);
  }
}
```

## 説明
- 未ログイン時(`pubkey` 無し)は即 `defaultRelays` を適用する。
- ログイン済みで `lumiSetting.useRelaySet !== "0"` の場合、ユーザのローカル設定 `lumiSetting.relays` をそのまま使う。
- `useRelaySet === "0"` の場合は `relaySearchRelays` をブートストラップに使って kind:10002 をフェッチし、`setRelaysByKind10002` でユーザのリレーセットを適用する。
- kind:10002 が見つからない場合は `defaultRelays` にフォールバックする。
- kind:10002 / kind:3 のパースは `src/lib/stores/useRelaySet.ts` の `setRelaysByKind10002` / `setRelaysByKind3`(105-125行)で行い、旧形式は kind:3 content からも解釈する。

## 参考
- https://github.com/TsukemonoGit/lumilumi/blob/main/src/lib/components/renderSnippets/nostr/relay/SetDefaultRelays.svelte

---
← [README](../README.md)
