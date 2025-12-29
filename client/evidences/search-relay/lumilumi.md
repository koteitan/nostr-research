← [README](../README.md)

# Lumilumi 検索リレー

## 結論
- **検索リレー (NIP-50)**: relay.nostr.band, search.nos.today
- **ユーザー設定**: kind:10007 でカスタマイズ可能

## ソースコード

**ファイル**: `src/lib/func/constants.ts`

```typescript
export const nip50relays = [
  "wss://relay.nostr.band",
  "wss://search.nos.today",
  // "wss://relay.noswhere.com", // コメントアウト
  // "wss://bostr.nokotaro.com", // コメントアウト
  // "wss://nostr.wine",         // コメントアウト
];
```

**ファイル**: `src/routes/search/+page.svelte` (行 21, 44, 97-134)

```typescript
import { nip50relays } from "$lib/func/constants";

let searchRelays = $state(nip50relays);

const setSearchRelay = async () => {
  const data: string[] | undefined = queryClient.getQueryData([
    "searchRelay",
    lumiSetting.get().pubkey,
  ]);

  if (data) {
    searchRelays = data;
  } else {
    // kind:10007 (Search Relays List) を取得
    const fetchRelays = await usePromiseReq(
      {
        filters: [
          { authors: [lumiSetting.get().pubkey], kinds: [10007], limit: 1 },
        ] as Nostr.Filter[],
        operator: pipe(latest()),
      },
      undefined,
      undefined
    );

    if (fetchRelays.length > 0) {
      const relaylist = toGlobalRelaySet(fetchRelays[0].event);
      if (relaylist.length > 0) {
        searchRelays = relaylist;
      }
    }
  }
};
```

## 説明

- NIP-50 検索対応リレー: `relay.nostr.band`, `search.nos.today`
- ユーザーは kind:10007 (Search Relays List) でカスタム検索リレーを設定可能
- kind:10007 が存在する場合はそちらを優先
- 設定がない場合は `nip50relays` をデフォルトとして使用

## 注意
`relaySearchRelays` (directory.yabu.me, purplepag.es 等) はユーザープロフィール/リレーリスト検索用であり、NIP-50 ワード検索用ではない。

## 参考
- https://github.com/TsukemonoGit/lumilumi/blob/main/src/lib/func/constants.ts
- https://github.com/TsukemonoGit/lumilumi/blob/main/src/routes/search/+page.svelte

---
← [README](../README.md)
