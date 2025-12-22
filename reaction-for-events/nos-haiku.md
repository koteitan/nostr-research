← [README](../README.md)

# Nos Haiku リアクション取得方法

## 結論
- **リアクション取得方法**: バッチ取得 (1秒バッファ) + リアルタイム購読

## ソースコード

**ファイル**: `src/lib/resource.ts` (行 747-771)

```typescript
#fetchReaction = (event: NostrEvent) => {
    let filter: LazyFilter;
    const until = unixNow();
    if (isReplaceableKind(event.kind) || isAddressableKind(event.kind)) {
        const ap: nip19.AddressPointer = {
            ...event,
            identifier: isAddressableKind(event.kind) ? (getTagValue(event, 'd') ?? '') : ''
        };
        filter = {
            kinds: [7],
            '#a': [getCoordinateFromAddressPointer(ap)],
            limit: this.#limitReaction,
            until
        };
    } else {
        filter = { kinds: [7], '#e': [event.id], limit: this.#limitReaction, until };
    }
    let options: { relays: string[] } | undefined;
    const event10002: NostrEvent | undefined = this.getReplaceableEvent(10002, event.pubkey);
    if (event10002 !== undefined) {
        const relays = getInboxes(event10002).filter(this.#relayFilter).slice(0, this.#limitRelay);
        options = relays.length > 0 ? { relays } : undefined;
    }
    this.#rxReqB7.emit(filter, options);
};
```

**ファイル**: `src/lib/resource.ts` (行 125, 145, 208)

```typescript
#limitReaction = 500;

this.#rxReqB7 = createRxBackwardReq();

const bt: OperatorFunction<ReqPacket, ReqPacket[]> = bufferTime(this.#secBufferTime);  // 1000ms
const batchedReq7 = this.#rxReqB7.pipe(bt, batch(this.#mergeFilterRg));
```

**ファイル**: `src/lib/resource.ts` (行 1474-1492) - リアルタイム購読

```typescript
if (currentProfilePointer !== undefined) {
    filtersF.push({ kinds: [7], '#p': [currentProfilePointer.pubkey], since });
} else if (currentChannelPointer !== undefined) {
    filtersF.push({ kinds: [7], '#k': ['42'], '#e': [currentChannelPointer.id], since });
} else if (currentEventPointer !== undefined) {
    filtersF.push({ kinds: [7], '#e': [currentEventPointer.id], since });
} else if (currentAddressPointer !== undefined) {
    filtersF.push({
        kinds: [7],
        '#a': [getCoordinateFromAddressPointer(currentAddressPointer)],
        since
    });
} else if (isAntenna && followingPubkeys.length > 0) {
    // ...
    filtersF.push({ kinds: [7], '#p': followingPubkeys, since });
} else if (isTopPage) {
    filtersF.push({ kinds: [7], '#k': ['42'], since });
}
```

## 説明

### バッチ取得 (後方取得)
- イベント表示時に `#fetchReaction()` が呼ばれる
- rx-nostr の `createRxBackwardReq()` を使用
- `bufferTime(1000)` で1秒間リクエストをバッファ
- `batch()` でフィルタをマージしてバッチ送信
- `limit: 500` で最大500件取得
- `#e` タグ (通常イベント) または `#a` タグ (アドレス可能イベント) でフィルタ
- イベント作成者の NIP-65 リレー (inbox) を優先

### リアルタイム購読 (前方取得)
- ページ種別に応じた `since` 付きフィルタで継続購読
- プロフィールページ: `#p` タグで購読
- イベント詳細ページ: `#e` タグで購読
- チャンネルページ: `#e` + `#k` タグで購読
- アンテナ: フォロー中ユーザーの `#p` で購読

## 参考
- https://github.com/nikolat/nos-haiku/blob/main/src/lib/resource.ts

---
← [README](../README.md)
